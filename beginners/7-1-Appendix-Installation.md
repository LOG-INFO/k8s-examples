[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)

# 7-1. Appendix. Kubernetes Installation


## 1) Setting Physical Server
진행중

<details><summary>show</summary>
<p>
</p>
</details>

## 2) Create VM `( For Master )`
진행중

<details><summary>show</summary>
<p>
</p>
</details>


## 3) Docker, Kubernetes Installation

Ubuntu나 Debian등 다른 OS를 설치하시는 분들께서는 아래 공식싸이트에서 명령어 참고 바래요.
<br/>
<참고 URL> https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

### 3-1) Pre-Setting

<details><summary>show</summary>
<p>

### 3-1-1) SELinux 설정

아래 설정으로 SELinux을 permissive로 변경해야하고 

```sh
setenforce 0
```
리부팅시 다시 원복되기 때문에 아래 명령을 통해서 영구적으로 변경합니다 

```sh
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

아래 명령어로 결과 확인

```sh
sestatus
```


### 3-1-2) firewalld 비활성화
내용

```sh
systemctl stop firewalld
systemctl disable firewalld
systemctl disable NetworkManager
systemctl stop NetworkManager
```

### 3-1-3) 스왑 비활성화
스왑 사용시 kubelet이 실행되지 않음

```sh
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab
```

### 3-1-4) iptables 커널 옵션 활성화
net/bridge.bridge-nf-call-iptables 커널 옵션 활성화

```sh
sysctl -w net.bridge.bridge-nf-call-iptables=1
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
EOF
```


### 3-1-5) hosts 등록
계획된 master와 node의 호스트 이름과 IP를 모두 등록해줍니다.

```sh
cat << EOF >> /etc/hosts
192.168.0.30 k8s-master
192.168.0.31 k8s-node1
192.168.0.32 k8s-node2
EOF
```

### 3-1-6) 쿠버네티스 YUM 리포지토리 설정:
-

```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

</p>
</details>

### 3-2) Install 

<details><summary>show</summary>
<p>

### 3-2-1) Docker, Kubernetes 설치 

```sh
yum install -y docker kubelet kubeadm kubectl --disableexcludes=kubernetes

```

</p>
</details>



## 4) Clone VM `( For Node )`

### 4-1) Clone VM

<details><summary>show</summary>
<p>
</p>
</details>

### 4-2) Config Node

<details><summary>show</summary>
<p>
### 4-2-1) Network 변경하기

```sh
vi /etc/sysconfig/network-scripts/ifcfg-eth0
systemctl restart network
```

### 4-2-2) Host Name 변경

```sh
hostnamectl set-hostname k8s-node1
hostnamectl set-hostname k8s-node2
```

</p>
</details>


## 5) Initialize Master and Join Node

<참고 URL> 
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

### 5-1) Master

<details><summary>show</summary>
<p>


### 5-1-1) 초기화 명령 실행
`pod-network-cidr` 설명
<br/>
`apiserver-advertise-address` 설명
<br/>
실행 후 `[Your Kubernetes master has initialized successfully!]` 문구를 확인하고 아래 내용 복사해서 별도로 저장해 둡니다. 
<br/>
kubeadm join 192.168.0.30:6443 --token ki4szr.t3wondaclij6d1a3 \
    --discovery-token-ca-cert-hash sha256:2370f0451342c6e4bd0d38f6c2511bda5c50374c85e9c09da28e12dd666d5987

```sh
kubeadm init --pod-network-cidr=10.16.0.0/16 --apiserver-advertise-address=192.168.0.30
```

### 5-1-2) 환경변수 설정
root 계정을 이용해서 kubectl을 실행하기 위한 환경 변수를 설정

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 5-1-3) kubectl 자동완성 기능 설치
kubectl 사용시 [tab] 버튼을 이용해서 다음에 올 명령어 리스트를 조회 할 수 있어요.
<br/>
명령 실행 후 바로 적용이 안되기 때문에 접속을 끊고 다시 연결 후에 사용 가능합니다. 

```sh
yum install bash-completion -y
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

</p>
</details>


### 5-2) Node

<details><summary>show</summary>
<p>

### 5-2-1) IP 관련 설정
설명

```sh
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### 5-2-2) Node 연결
Master Init 후 복사 내용 붙여넣기

```sh
kubeadm join 192.168.0.30:6443 --token 7xd747.bfouwf64kz437sqs \
    --discovery-token-ca-cert-hash sha256:ec75641cd258f2930a7f73abfe540bb484eb295ad4500ccdaa166208f97c5117
```

### 5-2-3) Node 연결 확인
Master 서버에 접속해서 아래 명령 입력 후 결과 확인

```sh
kubectl get nodes
```

</p>
</details>

## 6) Add Plugin

### 6-1) Networking

Kubernetes Cluster Networking에는 많은 Plugin들이 있는데 그중 Calico 설치에 대한 내용 입니다.
<br/>
<참고 URL> https://kubernetes.io/docs/concepts/cluster-administration/networking/
<br/>
<참고 URL> https://docs.projectcalico.org/v3.9/getting-started/kubernetes/

<details><summary>show</summary>
<p>


### 6-1-1) Calico 설치
Calico는 기본 192.168.0.0/16 대역으로 설치가 되는데, 그럼  실제 VM이 사용하고 있는 대역대와 겹치기 때문에 수정을 해서 설치해야 할 경우

```sh
yum install wget
wget https://docs.projectcalico.org/v3.9/manifests/calico.yaml
sed s/192.168.0.0\\/16/10.16.0.0\\/16/g -i calico.yaml
kubectl apply -f calico.yaml
```

기본 대역으로 사용해도 문제 없을 경우

```sh
kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
```

</p>
</details>

### 6-2) Dashboard

아래 가이드는 내부망에서 Admin 유저가 모든 권한으로 Dashboard를 사용하기 위한 설치 내용 입니다.
강좌 실습을 위한 설정이니 실제 프로젝트에선 이렇게 사용하시면 안되요 ^^
<br/>
<참고 URL> https://github.com/kubernetes/dashboard

<details><summary>show</summary>
<p>

### 6-2-1) Dashboard 설치
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

### 6-2-2) 로그인시 skip 버튼 활성화
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

### 6-2-3) 권한부여
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

### 6-2-4) 백그라운드로 proxy 띄우기	
`--address`에 자신의 Host IP 입력 

```sh
nohup kubectl proxy --port=8001 --address=192.168.0.30 --accept-hosts='^*$' >/dev/null 2>&1 &
```

### 6-2-5) 접속 URL 

```sh
http://192.168.0.30:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.
```

</p>
</details>
