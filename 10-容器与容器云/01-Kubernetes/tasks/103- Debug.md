[TOC]

### 调试

#### Pod

~~~bash
$ kubectl describe pod pod-name    # POD运行过程
$ kubectl get pod pod-name -o yaml # 检查资源定义, 信息完整
$ kubectl logs pod-name [container-name] # 容器日志, Pod中仅包含一个容器，就可以忽略 container-name
$ kubectl exec -- it pod-name [--container main-app] -- /bin/bash # 进入POD中某个容器, 
# 如果只有一个那么可不加 --container 
~~~

dlv远程调试

目标机器安装dlv, 注意pod容器运行中,  go的编译需要禁止内联优化 go build -gcflags="all=-N -l" -o myapp main.go

~~~bash
$ dlv attach x-pid --listen=:7890 --headless=true --log=true
~~~

本地机器安装dlv, 源码目录下执行下面命令

~~~bash
$ dlv connect 10.211.55.101:7890
~~~

#### Service

~~~bash
$ kubectl describe svc svc-name
$ kubectl get svc svc-name -o yaml # 检查资源定义 type、targetPort[字符串、还是数字]、port、namespace、selector
$ kubectl get ep svc-name

# DNS服务是否正常
$ kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
~ cat /etc/resolv.conf
~ nslookup kubernetes.default
~ nslookup svc-name.default.svc.cluster.local 
~ nslookup svc-name-headless.default.svc.cluster.local
~ nslookup pod-name.svc-name-headless.default.svc.cluster.local
~~~

网络排查: 系统是否按照规定初始化、 kube-proxy、ipvs、tcpdump

~~~bash
$ ps auxw | grep kube-proxy
$ ipvsadm -ln

# 查下coredns的流量
$ tcpdump -nn -i flannel.1 host 10.211.55.61 and port 9153 -vv
~~~



