### Run App

k8s v.1.27.0

#### Depolyment

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  selector:
    matchLabels:		# 匹配的Pod标签
      app: nginx
  replicas: 2           # 2个Pod
  minReadySeconds: 10
  template:				# Pod模版
    metadata:
      labels:			# Pod标签
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - name:  http
          containerPort: 80
~~~

~~~bash
$ kubectl apply -f nginx-deploy.yaml
$ kubectl get deploy nginx-deploy -o yaml
$ kubectl get pods -l app=nginx

$ kubectl delete -f nginx-deploy.yaml
$ kubectl scale deploy/nginx-deploy --replicse=2
~~~

#### StatefuleSet

pv,pvc

~~~yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual  # 表示管理员手动创建持久化卷, 此模式,那么Pod就要固定在卷的创建节点了
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/var/lib/data"  # 先在node1创建
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual  # 一定和上面的一致
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
~~~

mysql: pod

~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:				
    type: Recreate		# 不实用滚动升级. 更新Pod会先停止
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
          # 在实际中使用 secret
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        # 挂载pvc   
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      # 声明pvc
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
      # 固定Pod到node1节点    
      nodeSelector:
      	kubenertes.io/hostname: node1
~~~

注意：不能对有状态应用规模进行扩缩. 下层的pv,只能绑定一个Pod. 

删除应用, 卷资源还需要管理员手动删除

~~~bash
$ kubectl delete -f mysql.yaml
$ kubectl delete -f mysql-pv.yaml
~~~
