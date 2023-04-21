[TOC]

### Deployment

Kubernetes v1.27.0

~~~bash
$ kubectl explain deploy
$ kubectl explain deploy.spec
$ kubectl explain deploy.spec.template.spec
~~~

#### Example

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  selector:
    matchLabels:		# 匹配的Pod标签
      app: nginx
  replicas: 2
  minReadySeconds: 10
  template:				# Pod模版
    metadata:
      labels:			# Pod标签
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        # 三种策略 Always(总是拉远程) Never(只使用本地)  IfNotPresent(优先本地, 无就拉取远程, 不配置默认的选项)
        imagePullPolicy: IfNotPresent
        ports:
        - name:  http		# 支持service的targetPort设置 http、80
          containerPort: 80
~~~

未设置的值都为默认值

操作

~~~bash
$ kubectl apply -f nginx-deploy.yaml
$ kubectl get deploy nginx-deploy -o yaml
# 版本变更,方便回滚
$ kubectl annotate deploy/nginx-deploy kubernetes.io/change-cause='update image to latest'

$ kubectl delete -f nginx-deploy.yaml
$ kubectl scale deploy/nginx-deploy --replicse=2
~~~

获取的deploy如下, 并不全, 可采用 kubectl explain deploy.spec.template.spec 获取到细节

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx-deploy","namespace":"default"},"spec":{"minReadySeconds":10,"replicas":2,"selector":{"matchLabels":{"app":"nginx"}},"template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"nginx:latest","imagePullPolicy":"IfNotPresent","name":"nginx","ports":[{"containerPort":80,"name":"http"}]}]}}}}
  creationTimestamp: "2023-04-18T03:27:32Z"
  generation: 3
  name: nginx-deploy
  namespace: default							# 默认default空间
  resourceVersion: "366610"
  uid: 8d79ce3b-bbe2-43b0-9672-180994bd0564
spec:
  minReadySeconds: 10
  progressDeadlineSeconds: 600
  replicas: 2									# 副本
  revisionHistoryLimit: 10						# 历史版本
  selector:										# Pod标签
    matchLabels:
      app: nginx
  strategy:										# 滚动更新策略
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:										# Pod模版
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:latest
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: "2023-04-18T03:27:32Z"
    lastUpdateTime: "2023-04-18T03:34:32Z"
    message: ReplicaSet "nginx-deploy-8f7885bf6" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2023-04-18T03:35:45Z"
    lastUpdateTime: "2023-04-18T03:35:45Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  observedGeneration: 3
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2
~~~

参考coredns的deplyement

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2022-08-06T13:21:18Z"
  generation: 13
  labels:
    k8s-app: kube-dns							# deploy标签
  name: coredns
  namespace: kube-system						# 空间
  resourceVersion: "354957"
  uid: c8df3ab3-b36d-4d9b-b365-f93552a4f799
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kube-dns							# 匹配的Pod标签
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 1
    type: RollingUpdate
  template:										# Pod模版
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: kube-dns						# Pod标签
    spec:
      containers:
      - args:
        - -conf
        - /etc/coredns/Corefile
        image: registry.aliyuncs.com/k8sxio/coredns:v1.8.6
        imagePullPolicy: IfNotPresent
        livenessProbe:							# 存活探针 tcpdump -nn -i cni0
          failureThreshold: 5
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        name: coredns
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        - containerPort: 9153
          name: metrics
          protocol: TCP
        readinessProbe:							# 可读性探针
          failureThreshold: 3
          httpGet:
            path: /ready
            port: 8181
            scheme: HTTP
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:								# 资源限制
          limits:
            memory: 170Mi
          requests:
            cpu: 100m
            memory: 70Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - all
          readOnlyRootFilesystem: true
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /etc/coredns
          name: config-volume
          readOnly: true
      dnsPolicy: Default
      nodeSelector:								# 默认调度
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: coredns
      serviceAccountName: coredns
      terminationGracePeriodSeconds: 30
      tolerations:								# 污点
      - key: CriticalAddonsOnly
        operator: Exists
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
      - effect: NoSchedule
        key: node-role.kubernetes.io/control-plane
      volumes:									# 挂载configMap作为配置
      - configMap:
          defaultMode: 420
          items:
          - key: Corefile
            path: Corefile
          name: coredns
        name: config-volume
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: "2022-08-06T14:07:17Z"
    lastUpdateTime: "2022-08-06T14:07:24Z"
    message: ReplicaSet "coredns-7dff7cbfd6" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2023-04-12T20:43:33Z"
    lastUpdateTime: "2023-04-12T20:43:33Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  observedGeneration: 13
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2
~~~

