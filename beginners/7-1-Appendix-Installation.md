[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)

# 7-1. Appendix. Kubernetes Installation


## 1~2) 한 대의 컴퓨터에 여러 VM을 만들 준비

-
<details><summary>show</summary>
<p>
</p>
</details>

## 3) VM 생성 ( Master 용 )

Kubernetes 설치 공식 페이지를 참조 했어요
<참고 URL> https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

<details><summary>show</summary>
<p>
</p>
</details>


## 4~6) 도커와 쿠버네티스 설치

-
<details><summary>show</summary>
<p>
</p>
</details>

## 7~10) VM 복사 및 CentOS 설정 ( Node 용 )

-
<details><summary>show</summary>
<p>
</p>
</details>

## 11~13) Master 초기화 및 Node 연결

-
<details><summary>show</summary>
<p>
</p>
</details>

## 14) Kubernetes Network와 Dashboard 설치

-
<details><summary>show</summary>
<p>
</p>
</details>

## 14-1) Network Plugin

Network Plugin에는 

<details><summary>show</summary>
<p>
</p>
</details>

## 14-2) Dashboard Plugin

아래 가이드는 내부망에서 Admin 유저가 모든 권한으로 Dashboard를 사용하기 위한 설치 내용 입니다.
강좌 실습을 위한 설정이니 실제 프로젝트에선 이렇게 사용하시면 안되요 ^^
<참고 URL> https://github.com/kubernetes/dashboard

<details><summary>show</summary>
<p>

### 1. Dashboard 설치
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

### 2. 로그인시 skip 버튼 활성화
아래 명령어로 Dashboard의 Edit 모드로 들어간 후에 

```sh
kubectl -n kube-system edit deployments.apps kubernetes-dashboard

```

args에 `- --enable-skip-login` 추가

```sh
-------------------------------
    spec:
      containers:
      - args:
        - --auto-generate-certificates
        - --enable-skip-login
-------------------------------
```

### 3. 권한부여
ClusterRoleBinding을 만들어서 Dashboard에서 전체 Object를 사용할 수 있도록 권한부여

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
`--address`에 자신의 Host IP 입력 

```sh
nohup kubectl proxy --port=8001 --address=192.168.0.30 --accept-hosts='^*$' >/dev/null 2>&1 &
```

### 5. 접속 URL 

```sh
http://192.168.0.30:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.
```

</p>
</details>
