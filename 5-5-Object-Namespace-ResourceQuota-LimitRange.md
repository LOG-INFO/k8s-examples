[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

# 5-5. Object-Namespace, ResourceQuota, LimitRange (Practice)

## Namespace

### Namespace (nm-1)
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-1
```

### Namespace (nm-2)
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-2
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-1
  labels:
    app: pod
spec:
  containers:
  - name: container
    image: tmkube/app
    ports:
    - containerPort: 8080
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
  namespace: nm-1
spec:
  selector:
     app: pod
  ports:
     - port: 9000
       targetPort: 8080
```


## ResourceQuota

### ResourceQuota	
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: nm-1
spec:
  hard:
    requests.memory: 3Gi
    limits.memory: 6Gi
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-1
  labels:
    app: pod
spec:
  containers:
  - name: container
    image: tmkube/app
    ports:
    - containerPort: 8080
```


## LimitRange

### LimitRange	
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-1
  namespace: nm-1
spec:
  limits:
  - type: Container
    min:
      memory: 1Gi
    max:
      memory: 4Gi
    defaultRequest:
      memory: 1Gi
    default:
      memroy: 2Gi
    maxLimitRequestRatio:
      memory: 3
```
