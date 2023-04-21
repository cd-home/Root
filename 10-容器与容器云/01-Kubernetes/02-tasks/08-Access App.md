### Access App

#### kubectl

~~~bash
$ kubectl get pods
~~~

#### client-go

~~~go
package main

import (
	"context"
	"fmt"
	"k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {
	// 在 .kube config 中使用当前上下文
	// path-to-kube config -- 例如 /root/.kube/config
	config, _ := clientcmd.BuildConfigFromFlags("", "/root/.kube/config")
	// 创建 clientSet
	clientSet, _ := kubernetes.NewForConfig(config)
	// 访问 API 以列出 Pod
	pods, _ := clientSet.CoreV1().Pods("default").List(context.TODO(), v1.ListOptions{})
	fmt.Printf("There are %d pods in the cluster\n", len(pods.Items))
}
~~~

#### port-forward

~~~bash
$ kubectl port-forward pod-x ip_node:ip_inner
$ kubectl port-forward pod-x :ip_inner
~~~

#### expose service

~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: whoami
spec:
  ports:
    - protocol: TCP
      name: web
      port: 80
  selector:
    app: whoami
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
  labels:
    app: whoami
spec:
  replicas: 2
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: containous/whoami
          ports:
            - name: web
              containerPort: 80
~~~

或者

~~~bash
$ kubectl expose deploy whoami --type=NodePort --name=whoami # NodePort ClusterIP None
~~~



