
![kubernetes](https://github.com/taemin77/k8s-examples/blob/master/github.JPG)

바로가기 : 
https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88#

# 2-1. Service (Practice)


## 1-1) ClusterIp

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
  - port: 80
    targetPort: 8080
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    svc: clusterip
spec:
  containers:
  - name: container
    image: kubetm/app
```

### Request Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: request-pod
spec:
  containers:
  - name: container
    image: kubetm/init
```

```sh
kubectl exec request-pod -it /bin/bash
```

nslookup

```sh
nslookup clusterip1
nslookup clusterip1.default.svc.cluster.local
```

curl

```sh
curl clusterip1/hostname
curl clusterip1.default.svc.cluster.local/hostname
```


## 1-2) Headless

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
    - port: 80
      targetPort: 8080    
  clusterIP: None
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod4
  labels:
    svc: headless
spec:
  hostname: pod-a
  subdomain: headless1
  containers:
  - name: container
    image: kubetm/app
```

Nslookup


```sh
nslookup headless1
nslookup pod-a.headless1
nslookup pod-b.headless1

```
Curl

```sh
curl pod-a.headless1:8080/hostname
curl pod-b.headless1:8080/hostname
```

## 2-1) Endpoint1


### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: endpoint1
spec:
  selector:
    svc: endpoint
  ports:
  - port: 8080
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod7
  labels:
    svc: endpoint
spec:
  containers:
  - name: container
    image: kubetm/app
```


```sh
kubectl describe endpoints endpoint1
```

## 2-2) Endpoint2


### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: endpoint2
spec:
  ports:
  - port: 8080
```

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod9
spec:
  containers:
  - name: container
    image: kubetm/app
```

### Endpoint
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: endpoint2
subsets:
 - addresses:
   - ip: 20.111.156.74
   ports:
   - port: 8080
```

```sh
curl endpoint2:8080/hostname
```

## 2-3) Endpoint3


### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: endpoint3
spec:
  ports:
  - port: 80
```

Github의 Ip Address 확인

```sh
nslookup https://www.github.com
curl -O 185.199.110.153:80/taemin77/k8s-examples/blob/master/beginners/5-1-Object-Pod-Practice.md
```

### Endpoint
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: endpoint3
subsets:
 - addresses:
   - ip: 185.199.110.153
   ports:
   - port: 80
```

```sh
curl -O endpoint3/taemin77/k8s-examples/blob/master/beginners/5-1-Object-Pod-Practice.md
```




## 3) ExternalName

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
 name: externalname1
spec:
 type: ExternalName
 externalName: github.github.io
```

```sh
curl -O externalname2/taemin77/k8s-examples/blob/master/beginners/5-1-Object-Pod-Practice.md
```


