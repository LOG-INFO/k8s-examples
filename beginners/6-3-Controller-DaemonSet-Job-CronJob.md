[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)

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

## 3) CronJob
 
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