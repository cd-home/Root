[TOC]

### Debug/Loggin

#### Debug

资源是否存在、资源描述是否有问题、资源启动的日志

##### Pod

~~~bash
$ kubectl describe pod pod-name    # POD运行过程
$ kubectl get pod pod-name -o yaml # 检查资源定义, 信息完整
$ kubectl logs pod-name [container-name] # 容器日志, Pod中仅包含一个容器，就可以忽略 container-name
$ kubectl exec -- it pod-name [--container main-app] -- /bin/bash # 进入POD中某个容器, 
# 如果只有一个那么可不加 --container 
~~~

观察Pod创建调度的过程、每个阶段的状态描述; 例如, 资源、镜像无法拉取问题等

dlv远程调试

目标机器安装dlv, 注意pod容器运行中,  go的编译需要禁止内联优化 go build -gcflags="all=-N -l" -o myapp main.go

~~~bash
$ dlv attach x-pid --listen=:7890 --headless=true --log=true
~~~

本地机器安装dlv, 源码目录下执行下面命令

~~~bash
$ dlv connect 10.211.55.101:7890
~~~

##### Service

~~~bash
$ kubectl describe svc svc-name
$ kubectl get svc svc-name -o yaml # 检查资源定义 type、targetPort[字符串、还是数字]、port、namespace、selector
$ kubectl get ep svc-name
~~~

网络排查: 系统是否按照规定初始化、 kube-proxy、coredns  服务是否正常, 排查ipvs、tcpdump

~~~bash
$ kubectl -nkube-system get pod
$ ps auxw | grep kube-proxy    # 主要错误可能是没有conntrack
$ kubectl -nkube-system get pod kube-proxy-x -o yaml # 查看配置, mode是否是ipvs
$ kubectl -nkube-system logs kube-proxy-x
$ ps auxw | grep coredns       # 网络插件是否正常,如Flannel
$ ipvsadm -ln

# DNS服务是否正常, service 是否能通过dns访问、是否能通过ip访问
$ kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
~ cat /etc/resolv.conf
~ nslookup kubernetes.default
~ nslookup svc-name.default.svc.cluster.local 
~ nslookup svc-name-headless.default.svc.cluster.local
~ nslookup pod-name.svc-name-headless.default.svc.cluster.local

# 查下coredns的流量
$ tcpdump -nn -i flannel.1 host 10.211.55.61 and port 9153 -vv
# 查网桥流量
$ tcpdump -nn -i cni0
~~~

在排查是否存在针对Pod的网络策略

#### Loggin

~~~bash
# Coontainers日志
/var/log/containers

# Pod日志
/var/log/pods

# kubelet日志
/var/log/messages
~~~

