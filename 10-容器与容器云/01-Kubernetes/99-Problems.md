### Problems

namespace terminating 无法删除

~~~bash
$ nohup kubectl proxy --port=8009
$ ns=kubernetes-dashbord
$ curl -X PUT 
--data-binary @<(kubectl get namespace $ns -o json | sed 's/"kubernetes"//g')     
-H "Content-Type: application/json"     http://127.0.0.1:8009/api/v1/namespaces/$ns/finalize
~~~

