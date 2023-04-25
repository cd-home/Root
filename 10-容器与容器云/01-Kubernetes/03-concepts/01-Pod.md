[TOC]

### Pod

**Pod** 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元. Pod是容器运行的环境.  Pod是一种逻辑概念. 

#### Pod Example

Example

~~~yaml
apiVersion: v1
kind: Pod
metadata:  
  name: nginx  
  namespace: default
  labels:    
    app: nginx
spec:  
  containers:  
  - name: nginx    
    image: nginx    
    ports:    
    - containerPort: 80
~~~

Pod 通常不是直接创建的, 而是使用工作负载资源创建的, 例如Deployment、Job、Daemonset、Statefulset等.  

~~~bash
$ kubectl apply -f nginx-pod.yaml
~~~

Pod的更新一般涉及: spec.containers[*].image等, metadata与namespace一般是不可变的.  

#### Pod Principle

Pod 是一组紧密关联的容器集合, 它们共享 PID、IPC、Network 和 UTS Namespace, 是Kubernetes 调度的基本单位. Pod 的设计理念是支持多个容器在一个 Pod 中共享网络和文件系统, 可以通过进程间通信和文件共享这种简单高效的方式组合完成服务. 容器本质上就是进程, 那么 Pod 实际上就是进程组, 一组进程是作为一个整体来进行调度的.

##### 网络与存储

Pod 天生地为其成员容器提供了两种共享资源: **网络和存储**

每个 Pod 都在每个地址族中获得一个唯一的 IP 地址.  Pod 中的每个容器共享网络名字空间, 包括 IP 地址和网络端口. **Pod 内**的容器可以使用 `localhost` 互相通信.

一个 Pod 可以设置一组共享的存储卷. 

![Pod-Network](./images/Pod-Network.svg)

Pod中 A、B容器通过共享Infra容器的网络空间

Pod声明Volume, A、B也同时声明挂载该卷.  Kubernetes 中一个非常重要的设计模式：sidecar 模式的常用方式. 典型的场景就是容器日志收集. 

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
  namespace: default
spec:
  # Pod挂载hostPath主机目录
  volumes: 
  - name: varlog
    hostPath: 
      path: /var/log/counter
  containers:
  # app 
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/app.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  # sidecar    
  - name: count-log
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/app.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
~~~

##### 生命周期

###### Pod status

Pending:  Pod信息已经提交, 未被调度, 或者镜像正在Pull; Running: Pod至少有一个容器已经运行; Successded: 所以容器都被终止, 不会再重启. Failed: Pod中存在容器因失败终止. Unknow: 目前无法获取Pod状态. 

![Pod-Life](./images/Pod-Life.svg)

###### Init Container

Pod中最先启动的容器, 如果 Pod 的 Init 容器失败, kubelet 会不断地重启该 Init 容器直到该容器成功为止,  Init 容器不支持 `lifecycle`、`livenessProbe`、`readinessProbe` 和 `startupProbe`， 因为它们必须在 Pod 就绪之前运行完成.

在 Pod 启动过程中, 每个 Init 容器会在网络和数据卷初始化之后按顺序启动

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-nginx
  namespace: default
spec:
  volumes:
  - name: html
    emptyDir: {}
  initContainers:
  - name: init
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "echo 'Hello K8s Nginx Pod!' > /html/index.html"]
    volumeMounts:
    - name: html
      mountPath: "/html"
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
~~~

Tips: volumes挂载emptyDir: {} 是一个临时目录.  数据会保存在 kubelet 的工作目录下面, 生命周期等同于 Pod 的生命周期

~~~bash
$ kubectl apply -f nginx-init.yaml
$ kubectl delete -f nginx-init.yaml
$ kubectl logs init-nginx web
~~~

###### Pod Hook

容器生命周期钩子; Pod Hook 是由 kubelet 发起的, 当容器中的进程启动前或者容器中的进程终止之前运行, 包含在容器的生命周期之中

1. `PostStart`：这个钩子在容器创建后立即执行
2. `PreStop`：这个钩子在容器终止之前立即被调用

有两种方式来实现上面的钩子函数：

1. Exec - 用于执行一段特定的命令, 不过要注意的是该命令消耗的资源会被计入容器
2. HTTP - 对容器上的特定的端点执行 HTTP 请求

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-start-hook
spec:
  containers:
  - name: nginx-start-hook
    image: nginx
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
~~~

~~~bash
$ kubectl exec -it nginx-pod-hook -- cat /usr/share/message
~~~

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-stop-hook
  namespace: default
spec:
  volumes:
  - name: message
    hostPath:
      path: /tmp
  containers:
  - name: nginx-stop-hook
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: message
      mountPath: /usr/share/
    lifecycle:
      preStop:
        exec:
          command: ['/bin/sh', '-c', 'echo Hello from the preStop Handler > /usr/share/message']
~~~

###### Pod Probe

健康检查: liveness probe（存活探针`）和`readiness probe（可读性探针）

支持下面几种配置方式

1. exec: 执行一段命令
2. http: 检测某个 http 请求 (例如coredns)
3. tcpSocket: 使用此配置, kubelet 将尝试在指定端口上打开容器的套接字. 如果可以建立连接, 容器被认为是健康的.

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod-liveness
spec:
  containers:
  - name: busybox-pod
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600"]
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5 # 第一次探测的延迟时间, 一般根据应用来看
      periodSeconds: 5       # 间隔时间
  
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/liveness # 镜像拉取不了
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
~~~

返回200-400之间的响应码认为是成功的, 否则kubelet会将它杀死, 重启. 

###### Pod Resources

容器启动之前还会为当前的容器设置分配的 CPU、内存等资源

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-resource
spec:
  containers:
  - name: nginx-resource
    image: nginx
    ports:
    - containerPort: 80
    resources:
      requests:       # 集群资源要求
        memory: 50Mi
        cpu: 50m
      limits:         # 最大限制配置
        memory: 100Mi
        cpu: 100m
~~~

~~~
1 CPU = 1000 millicpu
1 MiB = 1024 KiB
~~~

50m = 0.05 CPU, 即使占5%CPU; 需要注意的是cpu是可压缩资源, 到达上限容器并不会退出. 内存到上线就会OOM.

###### Downward API

容器内部获取Pod的信息

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: downward-env-pod
  namespace: default
  labels:
    app: downward-env-pod
  annotations:
    version: v1
    build: prod
spec:
  # 挂载
  volumes:
  - name: podinfo
    downwardAPI:
      items:
      - path: labels
        fieldRef:
          fieldPath: metadata.labels
      - path: annotations
        fieldRef:
          fieldPath: metadata.annotations
  containers:
  - name: downward-env
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "env"]
    resources:
      requests:       
        memory: 50Mi
        cpu: 50m
      limits:         
        memory: 100Mi
        cpu: 100m
    # 通过env的方式注入
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: LIMITS_CPU
      valueFrom:
        resourceFieldRef:
          resource: limits.cpu
    - name: REQUESTS_MEM
      valueFrom:
        resourceFieldRef:
          resource: requests.memory
    volumeMounts:
      - name: podinfo
        mountPath: /etc/podinfo
~~~

#### Pod Spec

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  namespace: default
  # 标签
  labels:
    app: my-nginx
spec:
  containers:
  - name: my-nginx
    image: "docker.io/library/nginx:latest"
    # 三种策略 Always(总是拉远程) Never(只使用本地)  IfNotPresent(优先本地, 无就拉取远程, 不配置默认的选项)
    imagePullPolicy: IfNotPresent
    resources:
      # 限制资源
      limits:
        cpu: 100m
        memory: 100Mi
      # 要求的最小资源  
      requests:
        cpu: 100m
        memory: 100Mi
    # 容器暴露端口
    ports:
    - name:  http 
      containerPort:  80
  # 重启策略 Always OnFailure Never    
  restartPolicy: OnFailure
  # 节点选择
  nodeSelector:
  	kubernetes.io/hostname: node1
  # 污点
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
~~~

~~~bash
$ kubectl explain pod.spec
~~~

#### Static Pod

静态 Pod 通常绑定到某个节点上的kubelet. 其主要用途是运行自托管的控制面. 

静态 Pod(Static Pod)直接由特定节点上的 `kubelet` 守护进程管理, 不通过API Server. 尽管大多数 Pod 都是通过控制面(例如,Deployment)来管理的, 对于静态 Pod 而言, kubelet直接监控每个 Pod, 并在其失效时重启. 

通过配置文件的方式, kubelet会定期扫描

~~~bash
/etc/kubernetes/manifests
~~~

#### Commands

~~~bash
$ kubectl apply -f pod.yaml
$ kubectl delete -f pod.yaml
$ kubectl logs <pod name>
$ kubectl describe pod <pod name>
$ kubectl get pod <pod name> -o [wide|yaml] || --watch || -l app=nginx
$ kubectl get pod --show-labels
$ kubectl explain pod.spec
$ kubectl exec -it <pod name> -- /bin/sh
~~~
