[TOC]

### NetWork

#### Service NetWork

![k8s-network-service](./images/k8s-network-service.svg)

#### Service

A/AAAA

Cluster IP

返回Cluster IP VIP

~~~
my-svc.default.svc.cluster.local
my-svc.my-namespace.svc.cluster-domain.example
~~~

Headless, 返回pod ip集合

~~~
my-svc-headless.default.svc.cluster.local
my-svc-headless.my-namespace.svc.cluster-domain.example
~~~

#### Pod

A/AAAA

Cluster IP

~~~
pod-ip-address.default.pod.cluster.local
pod-ip-address.my-namespace.pod.cluster-domain.example
~~~

Headless 

~~~
pod-name.servicename.default.svc.cluster.local
pod-name.servicename.my-namespace.svc.cluster-domain.example
~~~

Hostname And Subdomain

~~~
pod-hostname.pod-subdomain.default.svc.cluster.local
pod-hostname.pod-subdomain.my-namespace.svc.cluster-domain.example
~~~

一些术语

NAT

SNAT

#### Why

modprobe br_netfilter

目的: 使得iptables规则在Linux Bridges上工作, 将桥接流量转发至iptables链

net.bridge.bridge-nf-call-iptables=1

目的: 表示 bridge 设备在二层转发时也去调用 iptables 配置的三层规则 (包含 conntrack)

影响: 如果没有开启, 那么会影响同一个node中的pod通过ClusterIP(DNS)通信

每个 Pod 的网卡都是 veth 设备, veth pair 的另一端连上宿主机上的cni0网桥

同一个node的pod之间流量走的是cni0网桥

Service同节点通信问题:

不论是iptables模式, 还是ipvs模式,  通过service访问都会是Cluster:IP(三层转发)  ==> DNAT ==> service[POD:IP]

然后通过conntrack记录连接, 以便在回包时SNAT

但是回包的时候, 发现目的pod就在同一个node上, 导致走cni0网桥, 不走三层转发(没有原路返回)
