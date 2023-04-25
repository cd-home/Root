[TOC]

### StatefulSet

管理有状态应用的工作负载 API 对象.  **StatefulSet 管理基于相同容器规约的一组 Pod**. 但和 Deployment 不同的是,  StatefulSet 为它们的**每个 Pod 维护了一个有粘性的 ID**. 这些 Pod 是基于相同的规约来创建的, 但是不能相互替换: 无论怎么调度, 每个 Pod 都有一个永久不变的 ID. 

有状态应用一般需要满足以下的条件:

1. **稳定的、唯一的网络标识符**
2. **稳定的、持久的存储**
3. 有序的、优雅的部署和扩缩
4. 有序的、自动的滚动更新

在为了满足以上条件:  需要

1. Headless Service 无头服务, 标识Pod唯一网络
2. PV, PVC 提供持久话存储
3. Statefulset提供的有序性部署、滚动更新

#### Example

##### PV

~~~yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ng-pv-x
  labels:
  	app: ng-pv
spec:
  capacity:
    storage: 100Mi
  accessModes:
  - ReadWriteOnce
  # hostath模式
  hostPath:
    path: /tmp/ng-pv-x
  # 亲和性
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node1
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ng-pv-y
  labels:
  	app: ng-pv
spec:
  capacity:
    storage: 100Mi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /tmp/ng-pv-y
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node2
~~~

注意这里只是演示使用PV: LocalPath存储模式, 正式环境不会如此使用. 

##### StatefulSet

~~~yaml
# 无头服务, 来控制网络域名
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
  namespace: default
  labels:
    app: nginx
spec:
  ports:
  - name: http
    port: 80
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
  namespace: default
spec:
  serviceName: "nginx-headless" # 即是StatefulSet的服务管理
  replicas: 2
  podManagementPolicy: OrderedReady  # 默认的管理策略 OrderedReady 并行策略 Parallel
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - name: web
          containerPort: 80
        volumeMounts:
        - name: ng-pvc-www
          mountPath: /usr/share/nginx/html
          readOnly: true
  # pvc卷申领模版        
  volumeClaimTemplates:
  - metadata:
      name: ng-pvc-www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 100Mi
      # 显式指定PV
      selector:
        matchLabels:
          type: ng-pv
  # pvc策略 v1.27 beta
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain
    whenScaled: Delete
  # 更新策略(可选)
  updateStrategy:
  	type: RollingUpdate # RollingUpdate(滚动更新), Ondelete(先手动删除Pod)
  	rollingUpdate:
  	  maxUnavailable:   # 最大不可用Pod
  	  partition:        # 分区滚动更新
~~~

###### Pod ID

分配稳定的0~n的id

###### Headless

无头服务:指的是ClusterIP: None, 不分配集群的IP, 提供稳定的网络访问Pod.

~~~bash
$ kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot:latest -- /bin/bash
~ nslookup nginx-headless.default.svc.cluster.local # 返回Pods的集合

~ ping nginx-headless.default.svc.cluster.local     # 某个固定的Pod, 不会随机轮训
~~~

还具有以下的特点, 由于Pod具有唯一的标识, 那么实际上Pod网络也有唯一标识. 

~~~bash
~ nslookup web-0.nginx-headless.default.svc.cluster.local
~ nslookup web-1.nginx-headless.default.svc.cluster.local
~~~

###### volumeClaimTemplates

pvc模版, pvc 被创建后会去关联当前系统中和他合适的 PV 进行绑定;  将pv的存储挂载到容器中, 会覆盖掉容器中的数据, 所以你需要手动的去pv里面创建index.html文件.

说明: 部署的时候是 -0 ~ -n 部署; 滚动更新的是n ~ 0;  如果指定了partition: x,  那么就是 x~0; 

删除的时候是n~0; 
