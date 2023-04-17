[TOC]

### Helm

#### Install

~~~bash
https://helm.sh/docs/intro/install/
helm repo add/remove/list/update
~~~

#### CMD

~~~bash
helm install redis ./redis -f ./redis/values.yaml 
helm delete redis
~~~

#### Service

##### MySQL 主从

~~~
LAST DEPLOYED: Tue Apr 11 16:58:36 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mysql
CHART VERSION: 9.7.1
APP VERSION: 8.0.32

** Please be patient while the chart is being deployed **

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace default

Services:

  echo Primary: mysql-primary.default.svc.cluster.local:3306
  echo Secondary: mysql-secondary.default.svc.cluster.local:3306

Execute the following to get the administrator credentials:

  echo Username: root
  MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run mysql-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mysql:8.0.32-debian-11-r21 --namespace default --env MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD --command -- bash

  2. To connect to primary service (read/write):

      mysql -h mysql-primary.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"

  3. To connect to secondary service (read-only):

      mysql -h mysql-secondary.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"
~~~

##### Redis 主从

~~~
[root@master master]# kubectl apply -f redis/storage-class.yaml 
[root@master master]# helm install redis ./redis/ -f ./redis/values-prod.yaml 
NAME: redis
LAST DEPLOYED: Thu Apr 13 01:04:48 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: redis
CHART VERSION: 17.9.3
APP VERSION: 7.0.10

** Please be patient while the chart is being deployed **

Redis&reg; can be accessed on the following DNS names from within your cluster:

    redis-master.default.svc.cluster.local for read/write operations (port 6379)
    redis-replicas.default.svc.cluster.local for read-only operations (port 6379)

To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default redis -o jsonpath="{.data.redis-password}" | base64 -d)

To connect to your Redis&reg; server:

1. Run a Redis&reg; pod that you can use as a client:

   kubectl run --namespace default redis-client --restart='Never'  \ 
   --env REDIS_PASSWORD=$REDIS_PASSWORD  \
   --image docker.io/bitnami/redis:7.0.10-debian-11-r4 \
   --command -- sleep infinity

   Use the following command to attach to the pod:

   kubectl exec --tty -i redis-client --namespace default -- bash

2. Connect using the Redis&reg; CLI: 修改了auth -a
   redis-cli -h redis-master -a REDIS_PASSWORD
   redis-cli -h redis-replicas -a REDIS_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/redis-master 6379:6379 &
    redis-cli -h 127.0.0.1 -p 6379 -a REDIS_PASSWORD
~~~

