
![kubernetes](https://github.com/taemin77/k8s-examples/blob/master/github.JPG)

바로가기 : 
https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88#

# 6-3. DaemonSet, Job, CronJob (Practice)

## 1-1) DaemonSet - HostPort
 
### DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-1
spec:
  selector:
    matchLabels:
      type: app
  template:
    metadata:
      labels:
        type: app
    spec:
      containers:
      - name: container
        image: tmkube/app
        ports:
        - containerPort: 8080
          hostPort: 18080
```

### Command
```sh
curl 192.168.0.31:18080/hostname
```

## 1-2) DaemonSet - NodeSelector
 
### DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-2
spec:
  selector:
    matchLabels:
      type: app
  template:
    metadata:
      labels:
        type: app
    spec:
      nodeSelector:
        os: centos
      containers:
      - name: container
        image: tmkube/app
        ports:
        - containerPort: 8080
```

### Kubectl
Label Add

```sh
kubectl label nodes k8s-node1 os=centos
kubectl label nodes k8s-node2 os=ubuntu
```
Label Remove

```sh
kubectl label nodes k8s-node2 os-
```


## 2) Job
 
### Job 1
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-1
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: container
        image: tmkube/init
        command: ["sh", "-c", "echo 'job start';sleep 20; echo 'job end'"]
        terminationGracePeriodSeconds: 0
```


### Job 2
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-2
spec:
  completions: 6
  parallelism: 2
  activeDeadlineSeconds: 30
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: container
        image: tmkube/init
        command: ["sh", "-c", "echo 'job start';sleep 20; echo 'job end'"]
        terminationGracePeriodSeconds: 0
```

## 3-1) CronJob
 
### CronJob
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: container
            image: tmkube/init
            command: ["sh", "-c", "echo 'job start';sleep 20; echo 'job end'"]
            terminationGracePeriodSeconds: 0
```

### Kubectl
Manual 

```sh
kubectl create job --from=cronjob/cron-job cron-job-manual-001
```
Suspend

```sh
kubectl patch cronjobs cron-job -p '{"spec" : {"suspend" : false }}'
```

## 3-2) CronJob - ConcurrencyPolicy 
(Allow, Forbid, Replace)
### CronJob
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cron-job-2
spec:
  schedule: "20,21,22 * * * *"
  concurrencyPolicy: Replace
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: container
            image: tmkube/init
            command: ["sh", "-c", "echo 'job start';sleep 140; echo 'job end'"]
            terminationGracePeriodSeconds: 0
```

