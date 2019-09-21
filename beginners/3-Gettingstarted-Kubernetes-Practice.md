
![kubernetes](https://github.com/taemin77/k8s-examples/blob/master/github.JPG)

바로가기 : 
https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88#

# 3. Getting-started Kubernetes (Practice)

## Linux

### hello.js
```javascript
var http = require('http');
var content = function(req, resp) {
 resp.end("Hello Kubernetes!" + "\n");
 resp.writeHead(200);
}
var w = http.createServer(content);
w.listen(8000);
```
```sh
node hello.js
```


## Docker 

### Dockerfile
```sh
FROM node:slim
EXPOSE 8000
COPY hello.js .
CMD node hello.js
```

### Docker Hub Site
https://hub.docker.com/

### Docker Container Run
```sh
docker build -t tmkube/hello .
-t : 레파지토리/이미지명:버전

docker images
docker run -d -p 8100:8000 tmkube/hello
-d : 백그라운드 모드
-p : 포트변경

docker ps
docker exec -it c403442e8a59 /bin/bash
```


### Docker Image Push
```sh
docker login
docker push tmkube/hello
```


## Kubernetes 

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    app: hello
spec:
  containers:
  - name: hello-container
    image: tmkube/hello
    ports:
    - containerPort: 8000
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-svc
spec:
  selector:
    app: hello
  ports:
    - port: 8200
      targetPort: 8000
  externalIPs:
  - 192.168.0.30
```
