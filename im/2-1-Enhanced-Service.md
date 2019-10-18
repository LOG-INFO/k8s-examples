
![kubernetes](https://github.com/taemin77/k8s-examples/blob/master/github.JPG)

바로가기 : 
https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88#

# 2-1. Enhance-Pod (Practice)


## 1) ClusterIp

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip1
spec:
  selector:
    svc: clusterip
  ports:
  - port: 8080
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-c1
  labels:
    svc: clusterip
spec:
  containers:
  - name: container
    image: tmkube/init
```


```sh
nslookup 172-16-4-158.default.pod.cluster.local
nslookup clusterip1.default.svc.cluster.local
nslookup clusterip1
```


## 2) Headless

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless1
spec:
  selector:
    svc: headless
  ports:
    - targetPort: 80
  clusterIP: None
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-h1
  labels:
    svc: headless
spec:
  hostname: pod1
  subdomain: headless1
  containers:
  - name: container
    image: tmkube/init
    ports:
    - containerPort: 80

```

```sh
mount | grep mount1
echo "file context" >> file.txt
```


## 2) Label

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
    type: web
    lo: dev
spec:
  containers:
  - name: container
    image: tmkube/init
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    type: web
  ports:
  - port: 8080
```


## 3) Node Schedule

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: tmkube/init

```

### Service
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
spec:
  containers:
  - name: container
    image: tmkube/init
    resources:
      requests:
        memory: 2Gi
      limits:
        memory: 3Gi
```
