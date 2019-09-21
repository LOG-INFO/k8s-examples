
![kubernetes](https://github.com/taemin77/k8s-examples/blob/master/github.JPG)

바로가기 : 
https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88#

# 6-2. Deployment (Practice)

## 1) ReCreate
 
### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  selector:
    matchLabels:
      type: app
  replicas: 2
  strategy:
    type: Recreate
  revisionHistoryLimit: 1
  template:
    metadata:
      labels:
        type: app
    spec:
      containers:
      - name: container
        image: tmkube/app:v1
      terminationGracePeriodSeconds: 10
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    type: app
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080

```

### Command
```sh
while true; do curl 10.99.5.3:8080/version; sleep 1; done
```

### Kubectl
```sh
kubectl rollout undo deployment deployment-1 --to-revision=2
kubectl rollout history deployment deployment-1
```

## 2) RollingUpdate

### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-2
spec:
  selector:
    matchLabels:
      type: app2
  replicas: 2
  strategy:
    type: RollingUpdate
  minReadySeconds: 10
  template:
    metadata:
      labels:
        type: app2
    spec:
      containers:
      - name: container
        image: tmkube/app:v1
      terminationGracePeriodSeconds: 0
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  selector:
    type: app2
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```

### Command
```sh
while true; do curl 10.99.5.3:8080/version; sleep 1; done
```



## 3) Blue/Green


### ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 2
  selector:
    matchLabels:
      ver: v1
  template:
    metadata:
      name: pod1
      labels:
        ver: v1
    spec:
      containers:
      - name: container
        image: tmkube/app:v1
      terminationGracePeriodSeconds: 0
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:
    ver: v1
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```
