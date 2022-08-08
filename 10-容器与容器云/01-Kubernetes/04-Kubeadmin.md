[TOC]

### Kubedm install K8s

#### Ready Arch

Linux Centos 8 ARM64 [2CPU, 4G, 60GB] [注意: 后续所有的操作针对 ARM64]

#### Ready Node 

- [x] nmcli 配置静态地址 [见Linux系统基础配置nmcli]

~~~bash
$ nmcli c modify enp0s5 connection.autoconnect yes
$ nmcli c modify enp0s5 ipv4.address 10.211.55.100/24
$ nmcli c modffy enp0s5 ipv4.method manul
$ nmcli c modify enp0s5 ipv4.gateway 10.211.55.1
$ nmcli c modify enp0s5 ipv4.dns '114.114.114.114'

$ nmcli c up enp0s5
~~~

- [x] hostnamectl 设置hostname

~~~bash
$ hostnamectl set-hostname master [node1 node2]
~~~

- [x] 修改 /etc/hosts

~~~bash
$ vim /etc/hosts
10.211.55.100  master
10.211.55.101  node1
10.211.55.102  node2
~~~

- [x] 关闭防火墙

~~~bash
$ systemctl stop firewalld
$ systemctl disable firewalld
~~~

- [x] 禁用 SELINUX (允许容器访问主机文件系统)

~~~bash
$ sestatus
# 临时
$ setenforce 0
# 永久
$ vim /etc/sysconfig/selinux
SELINUX=disable
# 可选操作
$ sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
# Need reboot
$ shutdown -r now
~~~

- [x] 关闭swap分区, 保证kubelet正常工作 [并且需要后续添加配置到k8s.conf]

~~~bash
$ swapoff -a
$ vim /etc/fstab
# 注释
#/dev/mapper/cl_fedora-swap none swap defaults 0 0
$ free -h
~~~

- [x] 允许 iptables 检查桥接流量

~~~bash
# 开启  br_netfilter
$ modprobe br_netfilter

$ vim /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness=0

$ sysctl -p /etc/sysctl.d/k8s.conf
# 重新加载
$ sysctl --system
~~~

- [x] 安装ipvs

~~~bash
$ cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
# 注意 高版本的centos内核nf_conntrack_ipv4被nf_conntrack替换
# 所以修改以上: modprobe -- nf_conntrack
$ bash /etc/sysconfig/modules/ipvs.modules
$ lsmod | grep -e ip_vs -e nf_conntrack
~~~

- [x] 安装ipset  ipvsadm

~~~bash
$ yum install ipset ipvsadm -y
~~~

- [x] 开启时钟同步

~~~bash
$ systemctl enable chronyd --now
# 手动同步
$ chronyc -a makestep
~~~

- [x] 安装容器运行时 containerd [1.24版本后将移除cri-dockerd]
    1. 见containerd二进制安装教程, 配置开机启动
    2. 注意配置sandbix_image, 以及镜像仓库地址[安装成功后可测试, 保证拉取镜像成功]

- [x] 配置 yum 源

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
~~~

- [x] 安装 [注意三个组件的版本最好保持一致]

~~~bash
$ yum install \
	kubeadm-1.24.3 \
	kubelet-1.24.3 \
	kubectl-1.24.3 -y \
	--disableexcludes=kubernetes
~~~

- [x] 检查端口是否被占用 6443 6379 等

#### Ready Cluster

- [x] 配置文件

~~~bash
$ kubeadm config print init-defaults > kubeadm.yaml

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
  # master IP
  advertiseAddress: 10.211.55.100
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: master
  # 配置master不运行普通pod
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
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
# 修改为阿里的镜像源, 拉取控制平面的容器
imageRepository: registry.aliyuncs.com/k8sxio
kind: ClusterConfiguration
kubernetesVersion: 1.24.3
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16  # flannel插件的网段
  serviceSubnet: 10.96.0.0/12
scheduler: {}
# 新增的
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
# 新增的, 配置 cgroupDriver 为 systemd
# 注意同样需要去containerd配置文件修改 SystemdCgroup = true
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
~~~

\>=1.22 版本情况下:

如果用户没有在 `KubeletConfiguration` 中设置 `cgroupDriver` 字段,  `kubeadm init` 会将它设置为默认值 `systemd`,所以上述的配置文件亦可不必手动填写. 

- [x] 先拉取镜像, 减少初始化的时间, 注意观察日志

~~~bash
$ kubeadm config images list
$ kubeadm config images pull --config kubeadm.yaml

# 注意: coredns 无法拉取, 手动拉取后打tag [imagePullPolicy: IfNotPresent] 不会重复拉取
$ ctr -n k8s.io i pull docker.io/coredns/coredns:1.8.6
$ ctr -n k8s.io i tag docker.io/coredns/coredns:1.8.6 \   registry.aliyuncs.com/k8sxio/coredns:v1.8.6
~~~

- [x] 初始化

~~~bash
$ kubeadm init --config kubeadm.yaml
~~~

过程中出现如下问题

1. 缺少 tc

~~~bash
$ yum install iproute-tc.aarch64
~~~

2. /proc/sys/net/bridge/bridge-nf-call-iptables does not exist

~~~bash
$ modprobe br_netfilter
$ echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
$ echo 1 > /proc/sys/net/ipv4/ip_forward
~~~

3. sandbox [修改containerd的配置文件 sandbox_image源] (导致无法创建容器)

~~~bash
$ vim /etc/containerd/config.toml

sandbix_image = "registry.aliyuncs.com/k8sxio/pause:3.6"
~~~

Successful

~~~bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.211.55.100:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:365171a0d3c85b57ec8e74cfebf06aba51267d30cee8e1f1de9e3666b0ddd9ae
# join会帮你启动kubelet, 然后你再开启开机启动即可

# Token 是有有效期的, 可以重新生成
# --token
$ kubeadm token create --print-join-command  
# --discovery-token-ca-cert-hash 
$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
~~~

- [x] 安装flannel网络插件, 并且修改 cni 配置

~~~bash
$ wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# 注意先修改里面的namespace: kube-system
$ kubectl apply -f kube-flannel.yml

# 修改名字, 让他读取 flannel 的配置 [貌似是按照字母读的配置]
$ mv /etc/cni/net.d/10-containerd-net.conflist \
/etc/cni/net.d/10-containerd-net.conflist.bak
~~~

Node节点可以先从Master把配置文件拿过来, 再加入集群,这样就pod就没必要删除重建.

注意

由于集群节点通常是按顺序初始化的, CoreDNS Pod 很可能都运行在第一个控制面节点上. 为了提供更高的可用性, 请在加入至少一个新节点后使用

~~~bash
$ kubectl -n kube-system rollout restart deployment coredns
~~~

更加暴力的做法可以删除pod, 让其自动重建

#### kubelet

~~~bash
$ kubectl get nodes
$ kubectl get pods -n [kube-system|default] [-o wide] [要注意命名空间]

$ kubectl describe pod 

$ kubectl get pods --watch -n default

$ kubectl apply -f nginx.yaml
$ kubectl delete -f nginx.yaml

$ kubectl explain x_deployment

$ kubectl get deployment [deploy]
~~~

部署nginx, 了解yaml

~~~yaml
apiVersion: apps/v1  # API版本
kind: Deployment     # API对象类型
metadata:			 # Map
  name: nginx-deploy
  labels:
    chapter: first-app
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2  		# Pod 副本
  template:    		# Pod 模板
    metadata:
      labels:
        app: nginx
    spec:
      containers:	# Array
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
~~~

转换为json

~~~json
{
    "apiVersion": "apps/v1",
    "kind": "Deployment",
    "metadata": {
      	"name": "nginx-deploy",
      	"labels": {
        	"chapter": "first-app"  
      }
    },
    "spec": {
        "selector": {
            "matchLabels": {
                "app": "nginx"
            }
        },
        "replicas": 2,
        "template": {
            "metadata": {
                "labels": {
                    "app": "nginx"
                }
            },
            "spec": {
                "containers": [
                    {
                        "name": "nginx",
                        "image": "nginx:latest",
                        "ports": [
                            {"containerPort": "80"}
                        ]
                    }
                ]
            }
        }
    }
}
~~~

对比看, yaml表达能力更好.

#### Tips

如何重新来过?

~~~bash
# 会提示删除必要的目录,配置等
$ kubeadm reset
$ iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
$ ipvsadm -C
~~~

