[TOC]

### Deployment

#### ReplicaSet 

ReplicaSet 确保任何时间都有指定数量的 Pod 副本在运行; 建议使用Deployment而不是使用ReplicaSet;

~~~yaml
apiVersion: apps/v1
kind: ReplicaSet  
metadata:
  name:  nginx-rs
  namespace: default
spec:
  replicas: 3  # 期望的 Pod 副本数量, 默认值为1
  selector:    # 匹配 Pod 模板中的标签
    matchLabels:
      app: nginx
  template:    # Pod 模板
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
~~~

~~~bash
$ kubectl apply -f nginx-rs.yaml
$ kubectl get rs
~~~

#### Deployment

ReplicaSet副本数量控制 + 滚动更新Rolling Update的能力; 无状态应用编排;

~~~yaml
apiVersion: apps/v1
kind: Deployment  
metadata:
  name:  nginx-deploy
  namespace: default
spec:
  replicas: 3     # 期望的 Pod 副本数量，默认值为1
  selector:       # Pod Label Selector
    matchLabels:
      app: nginx
  template:       # Pod 模板
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.26.0
        ports:
        - containerPort: 80
    # Deployment 中的容器restartPolicy=Always是唯一的, 也是默认的
~~~

~~~bash
$ kubectl apply -f nginx-deploy.yaml
$ kubectl get deploy
$ kubectl get rs 
$ kubectl get rs -w
~~~

Deployment 是通过管理 ReplicaSet 的数量和属性来实现`水平扩展/收缩`以及`滚动更新`两个功能

##### Scale

~~~bash
$ kubectl scale deply/nginx-deploy --replicas=2
~~~

##### Rolling update

.spec.template 发生改变

~~~yaml
apiVersion: apps/v1
kind: Deployment  
metadata:
  name:  nginx-deploy
  namespace: default
spec:
  replicas: 3  
  selector:  				# 指定deploy选择的 Pod, 必须匹配Pod模版的labels
    matchLabels:			# 创建后不可变
      app: nginx
  minReadySeconds: 30       # 最短就绪时间, 超出这个时间 Pod 才被视为可用
  revisionHistoryLimit: 10	# 历史版本(旧ReplicaSet), 默认是10
  strategy:  
    type: RollingUpdate     # 指定更新策略：RollingUpdate(滚动更新)、Recreate(全部删除, 重建)
    rollingUpdate:
      maxSurge: 1  	        # 创建的超出期望 Pod 个数的 Pod 数量, 支持百分比 25%
      maxUnavailable: 1     # 更新过程中不可用的 Pod 的个数上限, 支持百分比 25%
  template:  				# Pod模版
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.26.1
        ports:
        - containerPort: 80
~~~

~~~bash
$ kubectl apply -f nginx-deploy.yaml
$ kubectl annotate deploy/nginx-deploy kubernetes.io/change-cause='update image to 1.26.1'
$ kubectl get deploy -o wide|yaml
$ kubectl describe deploy nginx-deploy

# 查看滚动更新状态
$ kubectl rollout status deploy/nginx-deploy
# 暂停更新
$ kubectl rollout pause deploy/nginx-deploy
# 恢复更新
$ kubectl rollout resume deploy/nginx-deploy

# 版本控制
$ kubectl rollout history deploy nginx-deploy
$ kubectl rollout history deploy nginx-deploy --revision=2
$ kubectl rollout undo deploy nginx-deploy
$ kubectl rollout undo deploy nginx-deploy --to-revision=1
~~~

