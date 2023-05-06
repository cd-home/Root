[TOC]

### DaemonSet

**DaemonSet** 确保全部（或者某些）节点上运行一个 Pod 的副本. 当有节点加入集群时,  也会为他们新增一个 Pod . 当有节点从集群移除时, 这些 Pod 也会被回收. 删除 DaemonSet 将会删除它创建的所有 Pod.

DaemonSet 的一些典型用法: (例如kube-proxy, kube-flannel)

1. 在每个节点上运行集群守护进程
2. 在每个节点上运行日志收集守护进程
3. 在每个节点上运行监控守护进程

调度策略: DaemonSet 控制器为每个符合条件的节点创建一个 Pod, 并添加 Pod 的 `spec.affinity.nodeAffinity` 字段以匹配目标主机. Pod 被创建之后, 默认的调度程序通常通过设置 `.spec.nodeName` 字段来接管 Pod 并将 Pod 绑定到目标主机.

#### Example

~~~yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-ds
  namespace: default
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      # 容忍
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - name: http
          containerPort: 80
~~~

注意的是: 有某些节点可能污点, 所以需要在模版定义容忍.
