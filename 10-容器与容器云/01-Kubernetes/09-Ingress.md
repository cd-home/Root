### Ingress

Nginx-Ingress

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.0/deploy/static/provider/cloud/deploy.yaml
```

注意一下几点

需要两个镜像, 但是拉取有问题

```
registry.k8s.io/ingress-nginx/controller:v1.7.0
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230312-helm-chart-4.5.2-28
```

修改这两个镜像

~~~
docker.io/bitnami/nginx-ingress-controller:1.7.0
docker.io/dyrnq/kube-webhook-certgen:xxxx 这个是非"官方的"
~~~

将nginx-ingress-controller部署到固定可接入外网的节点, 需要修改

1. 给node打标签

~~~
kubectl label node k8s-node1 ingress-nginx=true
~~~

2. 节点选择

~~~
nodeSelector:
   kubernetes.io/os: linux
   ingress-nginx: "true"
~~~

注意这里一定是字符串的"true"

3. 采用 hostNetwork模式

~~~
spec:
	hostNetwork: true
~~~

使用 hostNetwork: true 进行部署 比 NodePort 减少一层转发