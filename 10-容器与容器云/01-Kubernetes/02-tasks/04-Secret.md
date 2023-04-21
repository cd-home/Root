[TOC]

### Secret

`Secret` 对象用来存储敏感数据

#### 指令

字面量字段

~~~bash
$ kubectl create secret generic my-secret --from-literal=db_user=root --from-literal=db_pwd='123456P'
~~~

文件

~~~bash
$ kubectl create secret generic my-secret --from-file=db_user='./user.txt' --from-file=db_pwd='./pwd.txt'
~~~

解码(被base64)

~~~bash
$ kubectl get secret my-secret -o jsonpath='{.data}' | base64 -d
~~~

编辑

~~~bash
$ kubectl edit secret my-secret
~~~

删除

~~~bash
$ kubectl delete secret my-secret
~~~

#### 声明

通过清单创建

~~~bash
echo -n 'admin' | base64  # YWRtaW4=
echo -n '123456' | base64 # MTIzNDU2
~~~

~~~yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MTIzNDU2
~~~

~~~bash
$ kubectl apply -f my-secret.yaml
~~~
