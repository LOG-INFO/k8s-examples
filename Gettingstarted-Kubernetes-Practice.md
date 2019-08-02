[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

# 3. Gettingstarted-Kubernetes-Practice

## linux

### hello.js
```bash
var http = require('http');
var content = function(req, resp) {
 resp.end("Hello Kubernetes!" + "\n");
 resp.writeHead(200);
}
var w = http.createServer(content);
w.listen(8000);
```

###node hello.js


## docker 

### Dockerfile
```bash
FROM node:slim
EXPOSE 8000
COPY hello.js .
CMD node hello.js
```


