[TOC]

### kubeadm

#### 重新配置集群

~~~bash
$ kubectl get cm -nkube-system

NAME                                                   DATA   AGE
coredns                                                1      46h
extension-apiserver-authentication                     6      46h
kube-apiserver-legacy-service-account-token-tracking   1      46h
kube-flannel-cfg                                       2      46h
kube-proxy                                             2      46h
kube-root-ca.crt                                       1      46h
kubeadm-config                                         1      46h
kubelet-config                                         1      46h

# kubelet配置
$ kubectl get cm -nkube-system kubelet-config -o yaml
# 集群配置
$ kubectl edit cm -nkube-system kubeadm-config
~~~

kubelet 配置修改后

1. 登录到 kubeadm 节点
2. 运行 `kubeadm upgrade node phase kubelet-config` 下载最新的 `kubelet-config` ConfigMap 内容到本地文件 `/var/lib/kubelet/config.conf`
3. 编辑文件 `/var/lib/kubelet/kubeadm-flags.env` 以使用标志来应用额外的配置
4. 使用 `systemctl restart kubelet` 重启 kubelet 服务

同理kube-proxy、coredns的修改通过configMap即可, 要反应更新, 可直接删除POD重建.

事实上一切皆资源、一切皆yaml.

~~~bash
$ kubectl get no node1
~~~

#### 管理证书

~~~bash
$ kubeadm certs check-expiration
$ kubeadm certs renew all
~~~

然后将/etc/kubernetes/manifests 清单移出, 默认0s, 然后移入

#### 管理内存、CPU、API资源

在namespace下, 创建属于该空间下的资源限制, 以此来同一管理POD

如配置命名空间下 Pod 配额, 限制该空间下的POD数量; 同理还有其他的LimitRange

~~~bash
$ kubectl create namespace quota-pod-example
$ vim rq.yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: pod-demo
      namespace: quota-pod-example
    spec:
      hard:
        pods: "2"
        
$ vim rq2.yaml
	apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: mem-cpu-demo
    spec:
      hard:
        requests.cpu: "1"
        requests.memory: 1Gi
        limits.cpu: "2"
        limits.memory: 2Gi
~~~

#### 清空Node, 维护升级

~~~bash
$ kubectl drain <node name>
$ kubectl uncordon <node name>
~~~

#### 调试DNS问题

~~~bash
# coredns以deploy启动
$ kubectl -nkube-system get deploy
$ kubectl get pods -nkube-system -l k8s-app=kube-dns
$ kubectl get svc --nkube-system
$ kubectl get endpoints --nkube-system kube-dns 

$ kubectl logs -nkube-system -l k8s-app=kube-dns
$ kubectl edit configmap -nkube-system coredns
$ kubectl describe clusterrole system:coredns -nkube-system

$ kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash

$ tmp-shell:~ cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5

$ tmp-shell:~ nslookup kubernetes.default
~~~

#### 名字空间

项目严格放置在归属的命名空间下, 做好资源的管理. 不同空间是不可见的.  并且namespace与DNS有关. 

~~~yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kubernetes-dashboard
~~~

注意, 删除namespace会删除下面所以的资源.

