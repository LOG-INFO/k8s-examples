
![kubernetes](https://github.com/taemin77/k8s-examples/blob/master/github.JPG)

바로가기 : 
https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88#

# 5-5. Object-Namespace, ResourceQuota, LimitRange (Practice)

## 1) Namespace
 
### Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-1
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

pod ip :

```sh
curl 10.16.36.115:8080/hostname
```
service ip :

```sh
curl 10.96.92.97:9000/hostname
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30000
  type: NodePort
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-2
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: tmkube/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: DirectoryOrCreate
```


```sh
echo "hello" >> hello.txt
```


## 2) ResourceQuota

### Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-3
```

### ResourceQuota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: nm-3
spec:
  hard:
    requests.memory: 1Gi
    limits.memory: 1Gi
```

ResourceQuota Check Command

```sh
kubectl describe resourcequotas --namespace=nm-3
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  containers:
  - name: container
    image: tmkube/app
    resources:
      requests:
        memory: 0.5Gi
      limits:
        memory: 0.5Gi
```


## 3) LimitRange

### Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-5
```

### LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-1
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.4Gi
    maxLimitRequestRatio:
      memory: 3
    defaultRequest:
      memory: 0.1Gi
    default:
      memory: 0.2Gi
```

LimitRange Check Command

```sh
kubectl describe limitranges --namespace=nm-5
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: tmkube/app
    resources:
      requests:
        memory: 0.1Gi
      limits:
        memory: 0.5Gi
```

### Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-6
```

### LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-5
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.5Gi
    maxLimitRequestRatio:
      memory: 1
    defaultRequest:
      memory: 0.5Gi
    default:
      memory: 0.5Gi
```

### LimitRange
```yaml
kind: LimitRange
metadata:
  name: lr-3
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.3Gi
    maxLimitRequestRatio:
      memory: 1
    defaultRequest:
      memory: 0.3Gi
    default:
      memory: 0.3Gi
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: tmkube/app
```
