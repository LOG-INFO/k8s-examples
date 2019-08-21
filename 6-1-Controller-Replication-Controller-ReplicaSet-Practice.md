[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

# 6-1. Controller-Replication Controller, ReplicaSet (Practice)

## 1. Template, Replicas
 
### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    type: web
spec:
  containers:
  - name: container
    image: tmkube/app:v1
  terminationGracePeriodSeconds: 0
```

### ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 1
  selector:
    matchLabels:
      type: web
  template:
    metadata:
      name: pod1
      labels:
        type: web
    spec:
      containers:
      - name: container
        image: tmkube/app:v1
      terminationGracePeriodSeconds: 0
```

## 2. Updating Controller
ReplicationController -> ReplicaSet

### ReplicationController
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: replication1
spec:
  replicas: 2
  selector:
    cascade: "false"
  template:
    metadata:
      labels:
        cascade: "false"
    spec:
      containers:
      - name: container
        image: tmkube/app:v1
```

### kubectl
```sh
kubectl delete replicationcontrollers replication1 --cascade=false
```

### ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica2
spec:
  replicas: 2
  selector:
    matchLabels:
      cascade: "false"
  template:
    metadata:
      labels:
        cascade: "false"
    spec:
      containers:
      - name: container
        image: tmkube/app:v1
```


## 3. Selector


### ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 1
  selector:
    matchLabels:
      type: web
      ver: v1
    matchExpressions:
    - {key: type, operator: In, values: [web]}
    - {key: ver, operator: Exists}
  template:
    metadata:
      labels:
        type: web
        ver: v1
        location: dev
    spec:
      containers:
      - name: container
        image: tmkube/app:v1
      terminationGracePeriodSeconds: 0
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-node-affinity1
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIngnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
  	       - {key: AZ-01, operator: Exists}
  containers:
  - name: container
    image: tmkube/init
```
