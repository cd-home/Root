[TOC]

### Jobs

#### Job

一次性任务, 批处理任务的一个或多个 Pod 成功结束

~~~yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-number
  labels:
  	app: "job"
spec:
  completions: 8			 # 需要完成多少个Pod
  parallelism: 2             # 并行执行个数
  activeDeadlineSeconds: 100 # 最大执行时间
  backoffLimit: 2            # 重试次数
  completionMode: Indexed    # job-number-1/2/3
  template:
    spec:
      restartPolicy: Never   # 只支持 Never OnFailure
      containers:
      - name: counter
        image: busybox:latest
        command:
        - "bin/sh"
        - "-c"
        - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done"
~~~

#### CronJob

周期性任务, 就是管理上述的一次性Job周期的执行

第1列分钟0～59、第2列小时0～23）、第3列日1～31、第4列月1～12、第5列星期0～7（0和7表示星期天）

~~~yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob-hello
  labels:
  	app: "cron-job"
spec:
  schedule: "* * * * *"
  successfulJobsHistoryLimit: 3  # 成功历史Pod保留
  failedJobsHistoryLimit: 3      # 失败历史Pod保留
  jobTemplate:					 # 即是Job的配置, 包含上述的完成、并发、重试、最大执行时间等
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:latest
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
~~~

~~~bash
$ kubectl get cronjob
$ kubectl get jobs --watch

$ kubectl delete cronjob hello
$ kubectl delete -f cronjob.yaml
~~~

