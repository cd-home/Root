### Inject Data

#### Command Args

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: temp-debian
  labels:
    app: temp-debian
spec:
  containers:
  - name: temp-debian
    image: debian:latest
    imagePullPolicy: IfNotPresent
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
~~~

执行shell

~~~yaml
command: ["/bin/sh"]
args: ["-c", "while true; do echo hello; sleep 10;done"]
~~~

#### Env

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: temp-debian
  labels:
    app: temp-debian
spec:
  containers:
  - name: temp-debian
    image: debian:latest
    imagePullPolicy: IfNotPresent
    env:
    - name: message
      value: 'k8s-2023'
    command: ["/bin/echo"]
    args: ["$(message)"]
~~~

如果要引用上文定义的环境变量, 必须是上文已经定义过的; 环境变量的顺序是重要的, 尤其是在依赖引用的情况下.

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: temp-debian
  labels:
    app: temp-debian
spec:
  containers:
  - name: temp-debian
    image: debian:latest
    imagePullPolicy: IfNotPresent
    env:
    - name: SERVICE_PORT
       value: "80"
    - name: SERVICE_IP
       value: "10.211.55.101"
    - name: PROTOCOL
      value: "http"
    - name: ADDRESS
      value: '$(PROTOCOL)://$(SERVICE_IP):$(SERVICE_PORT)'
~~~

通过卷访问secret, 并且通过挂载方式注入容器

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: temp-debian
  labels:
    app: temp-debian
spec:
  containers:
  - name: temp-debian
    image: debian:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
       - name: secret-volume
         mountPath: /etc/secret-volume-path
         readOnly: true
    command: ['/bin/bash']
    args: ['-c', "sleep 300"]
  volumes:
    - name: secret-volume
      secret:
        secretName: my-secret
        defaultMode: 0400
~~~

~~~bash
$ kubectl exec -it temp-debian -- /bin/bash
~ ls /etc/secret-volume-path
~~~

#### Env Ref Secret

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: temp-debian
  labels:
    app: temp-debian
spec:
  containers:
  - name: temp-debian
    image: debian:latest
    imagePullPolicy: IfNotPresent
    env:
    - name: db_user			   		# 环境变量名, 不一定要和secret的k一致
      valueFrom:
        secretKeyRef:
          name: my-secret           # 引用secret里面的k-v
          key: db_user
    - name: db_pwd			   	
      valueFrom:
        secretKeyRef:
          name: my-secret           # 可以是另一个secret
          key: db_pwd
    command: ['/bin/bash']
    args: ['-c', "sleep 300"]
~~~

