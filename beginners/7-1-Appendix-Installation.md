[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)

# 7-1. Appendix. Kubernetes Installation

## 1), 2) Host OS에 CentOS 설치 및 VM 가상화를 하기 위한 Virt-Manager 준비 

## 3) VM(Guest OS) 생성 및 CentOS 설치

## 4), 5), 6) CentOS 설정 및 Docker, Kubernetes 설치

## 7), 8) 1 Master와 2 Node를 구성하기 위해 VM 복사 

## 9), 10) CentOS 설정

## 11), 12), 13) Master 구성 및 Node 추가 

## 14) 네트워킹 구성, 대시보드 추가

## 14-1) Network Plugin

## 14-2) Dashboard Plugin

아래 가이드는 내부망에서 Admin 유저만 사용할 경우 [Kubectl Proxy]를 이용해서 모든 권한으로 Dashboard를 이용할 수 있는 설치 내용 입니다.

<details><summary>show</summary>
<p>

### 1. Dashboard 설치
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

### 2. 로그인시 skip 버튼 활성화
아래 명령어 Dashboard의 Edit 모드로 들어간 후에 args에 --enable-skip-login 추가


 
```sh
kubectl -n kube-system edit deployments.apps kubernetes-dashboard


-------------------------------
    spec:
      containers:
      - args:
        - --auto-generate-certificates
        - --enable-skip-login
-------------------------------
```

### 3. 권한부여
전체 Object 사용권한 부여

```sh
cat <<EOF | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
EOF	
```

### 4. 백그라운드로 proxy 띄우기	
--address에 자신의 Host IP 입력 (ex. 192.168.0.30)

```sh
nohup kubectl proxy --port=8001 --address=192.168.0.30 --accept-hosts='^*$' >/dev/null 2>&1 &
```

### 5. 접속 URL (ex. 192.168.0.30)
```sh
http://192.168.0.30:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.
```

</p>
</details>
