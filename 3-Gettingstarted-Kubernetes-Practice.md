[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

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
