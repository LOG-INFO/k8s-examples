
![kubernetes](https://github.com/taemin77/k8s-examples/blob/master/github.JPG)

바로가기 : 
<https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88#>

# 7-1. Appendix. Kubernetes Installation - Case 3

![install-case1](./images/install-case3.jpg)
<br/>
구글 클라우드 플랫폼을 이용해서 VM을 만드는 경우
<br/>
<br/>
<br/>


![install-8](./images/install-8.jpg)


## 1-1) GCP 가입

<details><summary>show</summary>
<p>


### 1-1-1) 접속

[윈도우10 버전으로 진행] 아래 경로에서 [Windows hosts] 클릭 하여 다운로드 후 설치 (별다른 변경없이 Next만 함)
<br/>
>https://www.virtualbox.org/wiki/Downloads

<br/>



</p>
</details>

<br/>
<br/>

![install-2](./images/install-2.jpg)

## 2-1) Setting VM

<details><summary>show</summary>
<p>



### 2-1-1) Master VM 생성
메모리 및 디스크는 각자 상황에 맞게 참고해서 VM 설정

```sh
11
```



### 2-1-3) 연결
Root 권한으로 변경

```sh
sudo su -
```

</p>
</details>

<br/>
<br/>

![install-3](./images/install-3.jpg)

## 3-1) Pre-Setting


<details><summary>show</summary>
<p>

### 3-1-1) SELinux 설정


Ubuntu나 Debian등 다른 OS를 설치하시는 분들께서는 아래 경로에서 명령어 참고 바래요
<br/>
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
<br/>
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker
<br/>
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


### 3-1-2) 방화벽 해제

firewalld 비활성화

```sh
systemctl stop firewalld && systemctl disable firewalld
```

NetworkManager 비활성화

```sh
systemctl stop NetworkManager && systemctl disable NetworkManager
```

### 3-1-3) Swap 비활성화
Swap 사용에 관련해서는 많은 의견이 있어요.
<br/>
>https://github.com/kubernetes/kubernetes/issues/53533
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

### 3-1-5) 쿠버네티스 YUM Repository 설정

YUM에 대해서 좀더 상세한 내용이 궁금한 분께서는 아래 싸이트가 잘 정리되어 있는거 같아 링크 첨부했어요.
<br/>
>https://www.lesstif.com/display/1STB/yum

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
계획된 master와 node의 호스트 이름과 IP를 모두 등록해주세요. 안하시면 추후 kubeadm init시 Host이름으로 IP를 찾을 수 없다고 에러가 나요.

```sh
cat << EOF >> /etc/hosts
192.168.0.30 k8s-master
192.168.0.31 k8s-node1
192.168.0.32 k8s-node2
EOF
```



</p>
</details>

## 3-2) Install 

<details><summary>show</summary>
<p>

### 3-2-1) Docker 설치 

도커 설치 전에 필요한 패키지 설치 

```sh
yum install -y yum-utils device-mapper-persistent-data lvm2 
```

 도커 설치를 위한 저장소 를 설정 

```sh
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

도커 패키지 설치 

```sh
yum update && yum install docker-ce-18.06.2.ce
```

```sh
mkdir /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

mkdir -p /etc/systemd/system/docker.service.d
```

### 3-2-2) Kubernetes 설치

```sh
yum install -y --disableexcludes=kubernetes kubeadm-1.15.5-0.x86_64 kubectl-1.15.5-0.x86_64 kubelet-1.15.5-0.x86_64
```

</p>
</details>



<br/>
<br/>

![install-4](./images/install-4.jpg)

## 4-1) Clone VM


<details><summary>show</summary>
<p>

### 4-1-1) 시스템 shutdown

여기까지 만든 이미지를 복사해 놓기 위해서 Master를 잠시 Shutdown 시켜요.

```sh
shutdown now
```

### 4-1-2) VM 복사

VirtualBox UI를 통해 Master 선택 후 마우스 우클릭을 해서 [복제] 버튼 클릭

```sh
1. 이름 : k8s-node1, MAC 주소정책 : 모든 네트워크 어댑터의 새 MAC 주소 생성
2. 복제방식 : 완전한 복제
```

node2도 반복


</p>
</details>

## 4-2) Config Node

<details><summary>show</summary>
<p>
   
### 4-2-1) Network 변경하기

VirtualBox UI에서 k8s-node1을 시작 시키면 뜨는 Console 창을 통해 아래 명령어 입력
<br/>
Host의 Ip Address를 변경하기 위해 아래 명령어로 설정을 열고

```sh
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
`IPADDR=` 부분을 해당 Node의 IP (192.168.0.31)로 변경해주세요

```sh
...
DEVICE="etho0"
ONBOOT="yes"
IPADDR="192.168.0.31"
...
```

그리고 아래 명령어로 네트워크 재시작

```sh
systemctl restart network
```

### 4-2-2) Host Name 변경
해당 Node의 Host 이름을 변경해주세요

```sh
hostnamectl set-hostname k8s-node1
```

이와 같은 방식으로 k8s-node2(192.168.0.32) 도 설정합니다.

</p>
</details>

<br/>
<br/>

![install-5](./images/install-5.jpg)

## 5-1) Master

<details><summary>show</summary>
<p>

### 5-1-1) 도커 및 쿠버네티스 실행
도커 실행

```sh
systemctl daemon-reload
```

```sh
systemctl enable --now docker
```

아래 명령어를 입력하면 image를 다운받는 내용이 나오면서 중간에  `Hello for Docker!` 가 보이면 설치 확인되면 설치가 잘 된거예요.

```sh
docker run hello-world
```

쿠버네티스 실행

```sh
systemctl enable --now kubelet
```


### 5-1-2) 쿠버네티스 초기화 명령 실행

kubeadm init 명령관련 해서 상세 내용이 궁금하신 분은 아래 싸이트 참고하세요.
>https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/
<br/>
`pod-network-cidr` 를 설정하면 Pod의 IP가 자동으로 생성될때 해당 network으로 생성되요

<br/>
`service-cidr` 를 설정하면 Service의 IP가 자동으로 생성될때 해당 대역으로 생성되요 `Default: 10.96.0.0/12`

```sh
kubeadm init --pod-network-cidr=20.96.0.0/12
```

실행 후 `[Your Kubernetes master has initialized successfully!]` 문구를 확인하고 아래 내용 복사해서 별도로 저장해 둡니다. 
<br/>
kubeadm join 192.168.0.30:6443 --token ki4szr.t3wondaclij6d1a3 \
    --discovery-token-ca-cert-hash sha256:2370f0451342c6e4bd0d38f6c2511bda5c50374c85e9c09da28e12dd666d5987
    
### 5-1-3) 환경변수 설정
root 계정을 이용해서 kubectl을 실행하기 위한 환경 변수를 설정

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 5-1-4) kubectl 자동완성 기능 설치
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


## 5-2) Node

<details><summary>show</summary>
<p>

### 5-2-1) 도커 및 쿠버네티스 실행

도커 실행

```sh
systemctl daemon-reload
```

```sh
systemctl enable --now docker
```

쿠버네티스 실행

```sh
systemctl enable --now kubelet
```

### 5-2-2) Node 연결
Master Init 후 복사 했었던 내용 붙여넣기

```sh
kubeadm join 192.168.0.30:6443 --token 7xd747.bfouwf64kz437sqs \
    --discovery-token-ca-cert-hash sha256:ec75641cd258f2930a7f73abfe540bb484eb295ad4500ccdaa166208f97c5117
```

### 5-2-3) Node 연결 확인
Master 서버에 접속해서 아래 명령 입력 후 추가된 Node가 보이는지 확인 (Status는 NotReady)

```sh
kubectl get nodes
```

</p>
</details>

<br/>
<br/>

![install-6](./images/install-6.jpg)

## 6-1) Networking

<details><summary>show</summary>
<p>


### 6-1-1) Calico 설치

Kubernetes Cluster Networking에는 많은 Plugin들이 있는데 그중 Calico 설치에 대한 내용 입니다.
<br/>
>https://docs.projectcalico.org/v3.9/getting-started/kubernetes/
<br/>
Calico는 기본 192.168.0.0/16 대역으로 설치가 되는데, 그럼  실제 VM이 사용하고 있는 대역대와 겹치기 때문에 수정을 해서 설치해야 할 경우

```sh
curl -O https://docs.projectcalico.org/v3.9/manifests/calico.yaml
sed s/192.168.0.0\\/16/20.96.0.0\\/12/g -i calico.yaml
kubectl apply -f calico.yaml
```

calico와 coredns 관련 Pod의 Status가 Running인지 확인 

```sh
kubectl get pods --all-namespaces
```

</p>
</details>

## 6-2) Dashboard


<details><summary>show</summary>
<p>

### 6-2-1) Dashboard 설치 

해당 설정은 교육목적으로 권한 설정을 모두 해제하는 방법이기 때문에 프로젝트에서 사용하실때는 이점 유의바래요
<br/>
>https://github.com/kubernetes/dashboard


```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

### 6-2-2) 권한 해지 설정 

접속시 인증 Skip 설정
<br/>
아래 명령을 통해 수정 모드로 들어가서

```sh
kubectl -n kube-system edit deployments.apps kubernetes-dashboard
```

 아래 내용 찾아서 `--enable-skip-login` 추가 

```sh
-------------------------------
    spec:
      containers:
      - args:
        - --auto-generate-certificates
        - --enable-skip-login
-------------------------------
```

Dashboard의 Admin권한 부여

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


### 6-2-3) 백그라운드로 proxy 띄우기	
`--address`에 자신의 Host IP 입력 

```sh
nohup kubectl proxy --port=8001 --address=192.168.0.30 --accept-hosts='^*$' >/dev/null 2>&1 &
```

### 6-2-4) 접속 URL 

```sh
http://192.168.0.30:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.
```

### 6-2-5) 언어 설정변경

```sh
Chrome 설정 > 언어 > 언어(한국어) > 원하는 언어를 위로 추가
```

</p>
</details>
