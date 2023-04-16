### Problems

namespace terminating 无法删除

~~~bash
$ nohup kubectl proxy --port=8009
$ ns=kubernetes-dashbord
$ curl -X PUT --data-binary @<(kubectl get namespace $ns -o json | sed 's/"kubernetes"//g') 
-H "Content-Type: application/json"     http://127.0.0.1:8009/api/v1/namespaces/$ns/finalize
~~~

网络调试

~~~bash
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
~~~

禁用ipv6

~~~bash
vim /etc/default/grub

GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX ipv6.disable=1"

grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg

ip a | grep inet6
~~~

<img src="/Users/liyao/WorkSpace/Root/10-容器与容器云/01-Kubernetes/images/kubectl.png" alt="kubectl" style="zoom:100%;" />

命令

~~~
kubectl rollout restart daemonset kube-proxy -n kube-system
kubectl scale deployment coredns --replicas=2 -n kube-system

kubectl edit cm coredns -nkube-system
kubectl exec -it --rm xx -- /bin/bash
~~~

抓包

~~~
ip a
tcpdump -i flannel.1 -vnn host xxx.xxx.xx.xx
tcpdump -i cni0 -vnn host xxx.xxx.xx.xx
~~~

常用命令

~~~
ip route show
~~~

DNS A/AAAA

A ipv4 AAAA ipv6

资源缩写

~~~
node		  		no
pod			  		po
deployment   		deploy
daemonset 			ds
service				svc
statefulset			sts
endpoint      		ep
namespace     		ns
persistentvolume    pv
persistentvolumeclaims pvc
storageclass        sc
networkpolicy		netpol
ingress				ing
configmap			cm
events				ev
replicaset 			rs
replicationcontroller rc
horizontalpodautoscaler hpa

customresourcedefinition crd

all
rolebinding
role
secret
clusterrolebinding
clusterrol
controllerrevision
cronjob
job
podpreset
podtemplates
~~~

查看信息

~~~bash
kubectl get -n xxnamespace xxtype -l k=v -o wide/yaml/json
~~~

版本发布

~~~
# 修改后 通过set或者apply 目前只是测试镜像改变有版本记录 nginx-deploy是资源的名字
kubectl set image deploy/nginx-deploy nginx=nginx:1.21.7
kubectl annotate deploy/nginx-deploy kubernetes.io/change-cause='update image to 1.21.7'

kubectl rollout history deploy nginx-deploy
kubectl rollout history deploy nginx-deploy --revision=2
kubectl rollout undo deploy nginx-deploy --to-revision=1

kubectl rollout restart deploy --selector=app=nginx
kubectl rollout restart deploy/nginx-deploy
~~~

修改yaml, kubectl set 的信息比较有限

~~~
kubectl edit deploy nginx-deploy -o --save-config [保存到annotation]
~~~

动态伸缩

~~~
kubectl scale deploy nginx-deploy --replicas=3
~~~

暴露服务

~~~
kubectl expose deploy nginx-deploy --port=8080 --target-port=80
~~~

通过声明的yaml文件更新应用(版本), 尽量不要使用命令行

查看日志

~~~
kubectl logs
~~~

查看系统支持

~~~
kubectl api-versions
kubectl api-resources
~~~

