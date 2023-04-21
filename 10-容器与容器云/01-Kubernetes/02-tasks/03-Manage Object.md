[TOC]

### Manage K8s Objects

kubectl(k8s) 支持: 指令式命令、指令式对象配置、声明式对象配置管理资源对象 v1.27.0

##### 声明式

Example

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  selector:
    matchLabels:		# 匹配的Pod标签
      app: nginx
  replicas: 2
  minReadySeconds: 10
  template:				# Pod模版
    metadata:
      labels:			# Pod标签
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        # 三种策略 Always(总是拉远程) Never(只使用本地)  IfNotPresent(优先本地, 无就拉取远程, 不配置默认的选项)
        imagePullPolicy: IfNotPresent
        ports:
        - name:  http		# 支持service的targetPort设置 http、80
          containerPort: 80
~~~

资源描述清单的字段不止这些, 如果没有显式声明, 那么就会给顶默认值

创建对象

~~~bash
$ kubectl create -f object.yaml
$ kubectl replace -f object.yaml
# 应用最终状态
$ kubectl apply -f object.yaml
~~~

默认会设置注解 kubectl.kubernetes.io/last-applied-configuration: {}

获取对象

~~~bash
$ kubectl get deploy nginx-deploy [-o wide] [-o yaml]
# 通过标签获取
$ kubectl get deploy -l name=nginx-o
# 查看描述
$ kubectl describe deploy nginx-deploy
~~~

删除对象

~~~bash
$ kubectl delete -f nginx-deploy.yaml
~~~

#### 指令式

运行Pod(创建Pod)

~~~bash
$ kubectl run nginx-pod --image=nginx:latest
~~~

暴露服务 (po, deploy, rs,rc)

~~~bash
# 给Pod创建服务
$ kubectl expose pod nginx-pod --port=8080 --target-port=80 --name=nginx-svc 
~~~

创建资源(svc,deploy,cm,secret,job,ingress,namespace等)

~~~bash
$ kubectl create service nodeport svc-pods --tcp=5678:8080
~~~

更新资源: 水平扩容、缩容

~~~bash
$ kubectl scale deploy/nginx-deploy --replicse=2
~~~

更新资源: 注解

~~~bash
# 注解: 版本变更,方便回滚
$ kubectl annotate deploy/nginx-deploy kubernetes.io/change-cause='update image to latest'
~~~

更新资源: 标签

~~~bash
$ kubectl label deploy/nginx-deploy name=nginx-o
~~~

更新资源:设置某些属性(有限)

~~~bash
$ kubectl set image deploy/nginx-deploy nginx=nginx:1.21.7
~~~

更新资源: 编辑清单

~~~bash
# --save-config  保存到annotation
$ kubectl edit deploy nginx-deploy -o --save-config 
~~~

补丁 TODO

~~~bash
$ kubectl patch
~~~

