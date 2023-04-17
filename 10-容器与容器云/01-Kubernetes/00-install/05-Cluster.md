[TOC]

### Kubernetes 高可用集群安装

#### 1. 系统初始化

准备5台Linux Centos Stream 9 Arm64, 设置五台节点主机名称、ip、网关、dns、/etc/hosts

~~~bash
10.211.55.60 k8s.cluster.vip.local
10.211.55.81 k8s-hm1  2CPU 4G 60G
10.211.55.82 k8s-hm2  2CPU 4G 60G
10.211.55.83 k82-hm3  2CPU 4G 60G
10.211.55.84 k8s-hn1  1CPU 2G 60G
10.211.55.85 k8s-hn2  1CPU 2G 60G	 
~~~

检查网络是否通、检查网卡情况

~~~bash
$ ip a
~~~

检查端口是否正常未被占用

控制面节点

| 协议 | 方向 | 端口范围  |          目的           |        使用者        |
| :--: | :--: | :-------: | :---------------------: | :------------------: |
| TCP  | 入站 |   6443    |  Kubernetes API server  |         所有         |
| TCP  | 入站 | 2379-2380 | etcd server client API  | kube-apiserver, etcd |
| TCP  | 入站 |   10250   |       Kubelet API       |     自身, 控制面     |
| TCP  | 入站 |   10259   |     kube-scheduler      |         自身         |
| TCP  | 入站 |   10257   | kube-controller-manager |         自身         |

Node节点

| 协议 | 方向 |  端口范围   |        目的        |    使用者    |
| :--: | :--: | :---------: | :----------------: | :----------: |
| TCP  | 入站 |    10250    |    Kubelet API     | 自身, 控制面 |
| TCP  | 入站 | 30000-32767 | NodePort Services† |     所有     |

配置yum源, 安装工具包, 更新源(注意, 采用的主机是Stream 9)

~~~bash
$ mv /etc/yum.repo.d/*repo /etc/yum.repo.d/backup

$ curl https://mirrors.tuna.tsinghua.edu.cn/epel/RPM-GPG-KEY-EPEL-9 > /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-9

$ vim /etc/yum.repos.d/CentOS-Stream9.repo
[BaseOS]
name=CentOS Stream $releasever - BaseOS
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos-stream/9-stream/BaseOS/aarch64/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-9
[Appstream]
name=CentOS Stream $releasever - AppStream
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos-stream/9-stream/AppStream/aarch64/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-9

$ yum install epel-release
$ yum install yum-utils
$ yum clean all && yum makecache
~~~

转发 IPv4 并让 iptables 看到桥接流量

~~~bash
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

$ sudo modprobe overlay
$ sudo modprobe br_netfilter

$ lsmod | grep br_netfilter
$ lsmod | grep overlay
~~~

~~~bash
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

$ sudo sysctl --system
~~~

关闭防火墙、禁用 SELINUX 、关闭Swap

~~~bash
$ systemctl stop firewalld
$ systemctl disable firewalld

$ sudo setenforce 0
$ sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

$ vim /etc/fstab # 注释如下行
#/dev/mapper/cl_fedora-swap none swap defaults 0 0
~~~

安装ipvs

~~~bash
$ cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
EOF

$ bash /etc/sysconfig/modules/ipvs.modules

$ $ yum install ipset ipvsadm -y
~~~

开启时钟同步

~~~bash
$ systemctl enable chronyd --now
$ chronyc -a makestep
~~~

#### 2. 容器运行时

1.24+ 全面启用containerd作为k8s容器运行时

安装cri-cni-containerd的组件 (见containerd安装, 目前安装版本1.7.0)

~~~bash
 containerd config default > /etc/containerd/config.toml
~~~

注意配置, cgroup采用systemd

~~~yaml
plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
~~~

注意sandbox_image, 修改为集群需要的版本, 以及(国内源)

~~~
[plugins."io.containerd.grpc.v1.cri"]
  # sandbox_image = "registry.k8s.io/pause:3.2"
  sandbix_image = "registry.aliyuncs.com/k8sxio/pause:3.9"
~~~

修改完成, 配置开机启动, 注意修改配置文件就需要重启

~~~bash
$ systemctl enable --now containerd.service
~~~

#### 3. 安装ETCD集群

~~~bash
10.211.55.81 k8s-hm1
10.211.55.82 k8s-hm2
10.211.55.83 k82-hm3
~~~

见etcd集群安装, cfssl制作证书

#### 4. kubeadm 引导安装

1. 安装kubeadm、kubelet、kubectl

~~~bash
$ cat > /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-aarch64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 默认安装 kubeadm、kubelet、kubectl 配套最新版 注意版本偏差
$ yum install kubeadmin

# 安装命令行补全
$ yum install bash-completion
$ echo 'source <(kubectl completion bash)' >>~/.bashrc
$ echo 'alias k=kubectl' >>~/.bashrc
$ echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
$ source ~/.bashrc
~~~

2. 配置kubeadm引导文件, 注意新增或者修改的部分

~~~bash
$ kubeadm config print init-defaults > kubeadm.yaml
~~~

~~~yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.211.55.81 							# 修改为当前master节点 k8s-hm1 IP
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: k8s-hm1												# 修改控制节点名称
  taints:													# 修改控制节点不参与调度POD
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
controlPlaneEndpoint: "k8s.cluster.vip.local:6443"			# 修改为kube-vip 虚拟VIP地址
apiServer:
  timeoutForControlPlane: 4m0s
  extraArgs:												# extra参数新增
    authorization-mode: Node,RBAC
  certSANs:  												# 新增添加其他master节点的相关信息
  - k8s.cluster.vip.local
  - k8s-hm1
  - k8s-hm2
  - k8s-hm3
  - 10.211.55.81
  - 10.211.55.82
  - 10.211.55.83
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:														# 修改采用外部拓扑ETCD
   external:
   endpoints:
     - https://10.211.55.81:2379 
     - https://10.211.55.82:2379 
     - https://10.211.55.83:2379 
   caFile: /etc/kubernetes/pki/etcd/ca.pem					# 证书由安装ETCD集群时2获得
   certFile: /etc/kubernetes/pki/etcd/etcd.pem
   keyFile: /etc/kubernetes/pki/etcd/etcd-key.pem
imageRepository: registry.aliyuncs.com/k8sxio				# 修改静态POD拉取地址
kind: ClusterConfiguration
kubernetesVersion: 1.27.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16									# 修改POD子网地址,采用Flannel插件
scheduler: {}
---															# 新增kube-proxy配置, 实际上后期可以去configMap编辑
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs  
---
apiVersion: kubelet.config.k8s.io/v1beta1					# 新增cgroup采用systemd, 并且kubelet是以服务启动的
kind: KubeletConfiguration
cgroupDriver: systemd
~~~

cgroup采用systemd说明, kubelet默认采用的是cgroupfs, 上述已经在kubeadm.yaml文件中新增. 实际上>1.22版本已经会默认添加.

可以去kubelet的配置文件中查看. 

原始配置参考如下

~~~yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.k8s.io
kind: ClusterConfiguration
kubernetesVersion: 1.27.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}

# 参考, kubeadm支持以下配置, 可参考官网
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kind: JoinConfiguration
~~~

3. 预先拉取镜像

~~~bash
$ kubeadm config images list
registry.k8s.io/kube-apiserver:v1.27.1
registry.k8s.io/kube-controller-manager:v1.27.1
registry.k8s.io/kube-scheduler:v1.27.1
registry.k8s.io/kube-proxy:v1.27.1
registry.k8s.io/pause:3.9						# 同步修改至containerd/config.toml sandbox_image
registry.k8s.io/etcd:3.5.7-0
registry.k8s.io/coredns/coredns:v1.10.1

$ kubeadm config images pull --config kubeadm.yaml

# 注意: coredns 无法拉取, 手动拉取后打tag [imagePullPolicy: IfNotPresent] 不会重复拉取
$ ctr -n k8s.io i pull docker.io/coredns/coredns:1.10.1
$ ctr -n k8s.io i tag docker.io/coredns/coredns:1.10.1 registry.aliyuncs.com/k8sxio/coredns:v1.10.1
~~~

4. kube-vip 

~~~bash
# https://kube-vip.io/docs/installation/static/
$ export VIP=10.211.55.60
$ export INTERFACE=enp0s5
$ export KVVERSION=v0.5.11

$ alias kube-vip="ctr image pull ghcr.io/kube-vip/kube-vip:$KVVERSION; ctr run --rm --net-host ghcr.io/kube-vip/kube-vip:$KVVERSION vip /kube-vip"

$ kube-vip manifest pod \
    --interface $INTERFACE \
    --address $VIP \
    --controlplane \
    --services \
    --arp \
    --leaderElection | tee /etc/kubernetes/manifests/kube-vip.yaml
    
 # 查看下kube-vip.yaml 保证信息准确 0.5.11
~~~

启动安装, 检测节点是否符合要求

~~~bash
$ kubeadm init --help
$ kubeadm init phase preflight --config kubeadm.yaml
$ kubeadm init --upload-certs  --config kubeadm.yaml

# 引导完成, 检测集群状态
$ kubectl cluster-info
~~~

可选参数参考

--pod-network-cidr=10.244.0.0/16 POD子网地址, 

--control-plane-endpoint 控制平面LB, 

--service-cidr= 10.96.0.0/12 集群服务子网

--cri-socket=unix:///var/run/containerd/containerd.sock (为empty, 会默认去找)

都已经在配置中了

注意参数 --upload-certs, 控制平面证书临时上传到集群中的 Secret, 2小时过期, 用于其他控制节点加入

~~~
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join k8s.cluster.vip.local:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:e443ecf3090e905abadc309e5aa187a72114d480d80b680129b4dc8f82a936d9 \
	--control-plane --certificate-key f57af1eeadf0a18502a90afed3f8a937d6acf95c3d66a1ee92bf3c85bb7720b1

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s.cluster.vip.local:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:e443ecf3090e905abadc309e5aa187a72114d480d80b680129b4dc8f82a936d9
~~~

5. 安装网络插件, 在安装之前, coredns POD没有启动

~~~bash
$ kubectl apply -f kube-flannel.yml
# 移走containd的网络配置
$ cd /etc/cni/net.d/ && mkdir ctn && mv /etc/cni/net.d/10-containerd-net.conflist ctn/
$ ls /etc/cni/net.d/
	10-flannel.conflist  ctn
# 查看cni0 是否是 10.244.x.x/24, 如果不是就要删除
$ ip a	
$ $ ifconfig cni0 down
$ ip link delete cni0
~~~

#### kube init ?

```bash
preflight                    Run pre-flight checks
certs                        Certificate generation
  /ca                          Generate the self-signed Kubernetes CA to provision identities for other Kubernetes components
  /apiserver                   Generate the certificate for serving the Kubernetes API
  /apiserver-kubelet-client    Generate the certificate for the API server to connect to kubelet
  /front-proxy-ca              Generate the self-signed CA to provision identities for front proxy
  /front-proxy-client          Generate the certificate for the front proxy client
  /etcd-ca                     Generate the self-signed CA to provision identities for etcd
  /etcd-server                 Generate the certificate for serving etcd
  /etcd-peer                   Generate the certificate for etcd nodes to communicate with each other
  /etcd-healthcheck-client     Generate the certificate for liveness probes to healthcheck etcd
  /apiserver-etcd-client       Generate the certificate the apiserver uses to access etcd
  /sa                          Generate a private key for signing service account tokens along with its public key
kubeconfig                   Generate all kubeconfig files necessary to establish the control plane and the admin kubeconfig file
  /admin                       Generate a kubeconfig file for the admin to use and for kubeadm itself
  /kubelet                     Generate a kubeconfig file for the kubelet to use *only* for cluster bootstrapping purposes
  /controller-manager          Generate a kubeconfig file for the controller manager to use
  /scheduler                   Generate a kubeconfig file for the scheduler to use
kubelet-start                Write kubelet settings and (re)start the kubelet
control-plane                Generate all static Pod manifest files necessary to establish the control plane
  /apiserver                   Generates the kube-apiserver static Pod manifest
  /controller-manager          Generates the kube-controller-manager static Pod manifest
  /scheduler                   Generates the kube-scheduler static Pod manifest
etcd                         Generate static Pod manifest file for local etcd
  /local                       Generate the static Pod manifest file for a local, single-node local etcd instance
upload-config                Upload the kubeadm and kubelet configuration to a ConfigMap
  /kubeadm                     Upload the kubeadm ClusterConfiguration to a ConfigMap
  /kubelet                     Upload the kubelet component config to a ConfigMap
upload-certs                 Upload certificates to kubeadm-certs
mark-control-plane           Mark a node as a control-plane
bootstrap-token              Generates bootstrap tokens used to join a node to a cluster
kubelet-finalize             Updates settings relevant to the kubelet after TLS bootstrap
  /experimental-cert-rotation  Enable kubelet client certificate rotation
addon                        Install required addons for passing Conformance tests
  /coredns                     Install the CoreDNS addon to a Kubernetes cluster
  /kube-proxy                  Install the kube-proxy addon to a Kubernetes cluster
show-join-command            Show the join command for control-plane and worker node
```

参考

1. kubeadm init phase preflight

2. kubeadm init phase kubelet-start

   kubelet 的配置会被写入磁盘 `/var/lib/kubelet/config.yaml`,  并上传到集群 `kube-system` 命名空间的 `kubelet-config` ConfigMap. kubelet 配置信息也被写入 `/etc/kubernetes/kubelet.conf`，其中包含集群内所有 kubelet 的基线配置

3. kubeadm init phase certs

   各种证书集群配置中心

4. kubeadm init phase kubeconfig

~~~bash
$ kubeadm init phase kubeconfig all

$ ls /etc/kubernetes/
	admin.conf  controller-manager.conf  kubelet.conf  manifests  pki  scheduler.conf
~~~

5. kubeadm init phase control-plane

   创建静态POD清单

~~~bash
$ kubeadm init phase control-plane all
$ ls /etc/kubernetes/manifests/
	kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml kube-vip.yaml
~~~

6. kubeadm init phase etcd (外部拓扑无此流程)

~~~bash
$ kubeadm init phase etcd local
~~~

7. kubeadm init phase upload-config

   kubeadm 配置上传到集群

~~~
kubeadm init phase upload-config [all,kubeadm,kubelet]
~~~

8. kubeadm init phase upload-certs

   使用以下阶段将控制平面证书上传到集群. 默认情况下, 证书和加密密钥会在两个小时后过期.

9. kubeadm init phase addon

   安装可用插件

~~~
kubeadm init phase addon [coredns,kube-proxy]
~~~

#### kubeadm reset?

~~~bash
kubeadm reset
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
ipvsadm -C
kubectl delete node --all
~~~

