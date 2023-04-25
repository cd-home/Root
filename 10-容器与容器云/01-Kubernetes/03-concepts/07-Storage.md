### Storage

内存资源的基本单位是字节(byte); 常用  Mi, Gi

#### HostPath

`hostPath` 卷能将主机节点文件系统上的文件或目录挂载到你的 Pod; hostPath 卷存在许多安全风险, 最佳做法是尽可能避免使用 hostPath. 当必须使用 HostPath 卷时, 它的范围应仅限于所需的文件或目录, 并以只读方式挂载.

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
        sleep 2;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
~~~

或者通过制备PV

~~~yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ng-pv-x
spec:
  capacity:
    storage: 100Mi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /tmp/ng-pv-x
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node1
~~~

问题有两点: 本地目录的存储行为是完全不可控的. 并且Pod使用该PV, 就必须绑定到该PV所在节点上. 

#### Local

`local` 卷所代表的是某个被挂载的本地存储设备, 例如磁盘、分区或者目录.  并且只能用作静态创建的持久卷, 不支持动态配置. 

与 `hostPath` 卷相比, `local` 卷能够以持久和可移植的方式使用, 而无需手动将 Pod 调度到节点. 

~~~yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ng-pv
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # Retain 保留 Recycle 回收 Delete 删除
  # 指定存储类
  storageClassName: local-storage
  # 磁盘、分区或者目录
  local:
    path: /mnt/disks/ssd1
  # 节点亲和性
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node1
~~~

需要注意的是: `local` 卷仍然取决于底层节点的可用性, 并不适合所有应用程序.  如果节点变得不健康, 那么 `local` 卷也将变得不可被 Pod 访问, 使用该卷 Pod 将不能运行. 所以使用 `local` 卷的应用程序必须能够容忍这种可用性的降低, 以及因底层磁盘的耐用性特征而带来的潜在的数据丢失风险. 

Kubernetes 调度器使用 PersistentVolume 的 `nodeAffinity` 信息来将使用 `local` 卷的 Pod 调度到正确的节点. 

本地卷还不支持动态制备, 所以使用 `local` 卷时,建议创建一个 StorageClass 并将其 `volumeBindingMode` 设置为 `WaitForFirstConsumer`. 延迟卷绑定的操作可以确保 Kubernetes 在为 PersistentVolumeClaim 作出绑定决策时, 会评估 Pod 可能具有的其他节点约束, 例如: 如节点资源需求、节点选择器、Pod 亲和性和 Pod 反亲和性. 

~~~yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
~~~





