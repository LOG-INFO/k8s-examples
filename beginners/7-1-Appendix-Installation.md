[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)

# 7-1. Appendix. Kubernetes Installation


## 1) Setting Physical Server
진행중

<details><summary>show</summary>
<p>
</p>
</details>

## 2) Create VM ( For Master )

CentOS 최신 버전 Minimal ISO 버전으로 설치
<참고 URL>
https://www.centos.org/download/

### 2-1) Setting VM

<details><summary>show</summary>
<p>



### 2-1-1) CentOS 최신버전 다운로드
Virt-Manager의 Image 파일 기본 경로로 이동

```sh
cd /var/lib/libvirt/images
```

다운로드! 만약 아래 주소가 없을 경우, 위 <참고 URL>에서 경로 다시 확인하시고 아래 명령어 수정이 필요합니다.

```sh
curl http://mirror.kakao.com/centos/7.7.1908/isos/x86_64/CentOS-7-x86_64-Minimal-1908.iso -O
```

### 2-1-2) Virt-Manager UI 설정
UI 실행 명령

```sh
virt-manager
```

6번 단계에서 `Host divice eno1`는 자신 서버에 물리 Port가 여러게 있을 경우, Port 갯수 만큼 생성되는데 선택한 Port로 해당 VM의 트래픽이 나가기 때문에 여러 VM을 만들때 분산해서 지정하면 좋아요

```sh
1. File > New Virtual Machine
2. [step1] Local install media (ISO image or CDROM) 선택 
3. [step2] Use ISO Image [Browse] 클릭해서 ISO 선택 
4. [step3] Memory(RAM) : 4096 MiB, CPUs 2로 변경 
5. [step4] 150 GiB 변경 
6. [step5] Name : k8s-master, [Network selection]을 Host divice eno1:mactab 선택 후 [Source mode]는 Bridge로 변경
7. Finish를 누르고 조금 기다리면 CentOS 설치 화면 나옴
```

</p>
</details>

### 2-2) Install CentOS

<details><summary>show</summary>
<p>

### 2-2-1) CentOS 설치

4번 단계에서 `8.8.8.8`는 Google DNS입니다. 원하는 DNS 쓰셔도 되요.

```sh
1. Test this media & install CentOS 7
2. Language : 한국어 
3. Disk 설정 [시스템 > 설치 대상]
   - [기타 저장소 옵션 > 파티션 설정] 파티션을 설정합니다. [체크] 후 [완료]
   - 새로운 CentOS 설치 > 여기를 클릭하여 자동으로 생성합니다. [클릭]
   - /home [클릭] 후 용량 5.12 GiB로 변경 [설정 업데이트 클릭]
   - / [클릭] 후 140 GiB 변경 후 [설정 업데이트 클릭]
   - [완료], [변경 사항 적용]
4. 네트워크 설정 [시스템 > 네트워크 및 호스트명 설정]
   - 호스트 이름: k8s-master [적용]
   - 이더넷 [켬], [설정], [IPv4 설정] 탭
   - 방식: 수동으로 선택, 
   - [Add] -> 주소: 192.168.0.30, 넷마스크 : 255.255.255.0, 게이트웨이: 192.168.0.1, DNS 서버 : 8.8.8.8 [저장]
5. 설치시작
6. [설정 > 사용자 설정] ROOT 암호 설정 
7. [재부팅]
```

</p>
</details>


## 3) Docker, Kubernetes Installation

Ubuntu나 Debian등 다른 OS를 설치하시는 분들께서는 아래 공식싸이트에서 명령어 참고 바래요
<br/>
<참고 URL> https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

### 3-1) Pre-Setting

<details><summary>show</summary>
<p>

### 3-1-1) SELinux 설정

쿠버네티스가 Pod Network에 필요한 호스트 파일 시스템에 액세스가 가능하도록 하기 위해서 필요한 설정이예요
<br/>
아래 설정으로 SELinux을 permissive로 변경해야하고 

```sh
setenforce 0
```
리부팅시 다시 원복되기 때문에 아래 명령을 통해서 영구적으로 변경 해야되요

```sh
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

아래 명령어를 실행해서 `Current mode:permissive` 내용 확인

```sh
sestatus
```


### 3-1-2) firewalld 비활성화

```sh
systemctl stop firewalld
systemctl disable firewalld
systemctl stop NetworkManager
systemctl disable NetworkManager
```

### 3-1-3) Swap 비활성화
Swap 사용에 관련해서는 많은 의견이 있어요.
<br/>
<참고 URL> https://github.com/kubernetes/kubernetes/issues/53533
<br/>
위 내용을 참고하셔서 swap 사용시의 고려해야할 점을 확인하시고 일단 여기선 사용하지 않도록 설정할께요.

```sh
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab
```

### 3-1-4) iptables 커널 옵션 활성화
RHEL이나 CentOS7 사용시 iptables가 무시되서 트래픽이 잘못 라우팅되는 문제가 발생한다고 하여 아래 설정이 추가되요

```sh
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 3-1-5) 쿠버네티스 YUM 리포지토리 설정

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

### 3-1-6) Centos Update

```sh
yum update
```

### 3-1-7) hosts 등록
계획된 master와 node의 호스트 이름과 IP를 모두 등록해줍니다.

```sh
cat << EOF >> /etc/hosts
192.168.0.30 k8s-master
192.168.0.31 k8s-node1
192.168.0.32 k8s-node2
EOF
```



</p>
</details>

### 3-2) Install 

<details><summary>show</summary>
<p>

### 3-2-1) Docker 설치 

```sh
yum install -y yum-utils device-mapper-persistent-data lvm2 docker-ce
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
systemctl enable docker && systemctl start docker
```

설치 확인

```sh
docker run hello-world
```

### 3-2-2) Kubernetes 설치 

```sh
yum install -y docker kubelet kubeadm kubectl --disableexcludes=kubernetes

```

</p>
</details>



## 4) Clone VM ( For Node )

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
