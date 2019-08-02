# 3. Gettingstarted-Kubernetes-Practice

### 

### Use label selectors to schedule Pods

<details>
<p>

```bash
$ kubectl label nodes node-1 size=Large
$ cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -	name: nginx
    image: nginx
  nodeSelector:
    size: Large

```
</p>
</details>
