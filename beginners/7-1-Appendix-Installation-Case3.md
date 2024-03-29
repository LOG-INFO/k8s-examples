
![kubernetes](https://github.com/taemin77/k8s-examples/blob/master/github.JPG)

바로가기 : 
<https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88#>

# 7-1. Appendix. Kubernetes Installation - Case 3

![install-case3](./images/install-case3-n.jpg)
<br/>
구글 클라우드 플랫폼을 이용해서 VM을 만드는 경우
<br/>
<br/>
<br/>


![install-10](./images/install-10.jpg)


## 1-1) Join GCP

<details><summary>show</summary>
<p>


### 1-1-1) Join

아래 사이트에 들어가서 상단에 [무료로 시작하기] 버튼 클릭
<br/>
>https://cloud.google.com/

<br/>

```sh
1) Country : South Korea
2) Terms of Service : 체크 후 [CONTINUE]
3) 이름 및 주소 확인 후 Payment method 입력
```

자신의 신용카드 번호를 입력하지만 크레딧을 다 소모하거나 12개월이 지나도 자동으로 결재가 되진 않습니다.

</p>
</details>

<br/>
<br/>

![install-11](./images/install-11.jpg)


## 2-1) Create Cluster

<details><summary>show</summary>
<p>



### 2-1-1) Cluster 생성

Kubernetes Engine > Clusters > [CREATE CLUSTER] 클릭

```sh
1. Name : k8s-cluster
2. Location Type : Zonal
3. Zone : asia-east1-a 부터 asia-east2-c 중 하나 선택
4. Master Version : 1.14.8-gke.2
5. Node pools > default-pool > Number of nodes : 2
6. Machine type : n1-standard-2 (2 vCPU, 7.5GB memory)
7. [Create] 클릭
```

</p>
</details>

![install-12](./images/install-12.jpg)

## 3-1) Connect GCP

<details><summary>show</summary>
<p>
 
 
### 3-1-1) Install SDK 
아래 URL에서 Windows용 GCP SDK 설치
<br/>

>https://cloud.google.com/sdk/docs/quickstart-windows

<br/>

다른 운영체제에서는 아래 내용 참조
<br/>

>https://cloud.google.com/sdk

```sh
1. [Google Cloud SDK 설치 프로그램] 다운로드
2. Next > Next 를 통해 설치 후 마지막에 4가지 체크항목 모두 선택 후 [Finish]
```
<br/>


## 3-2) Setting SDK Shell

### 3-2-1) Install SDK 
바탕화면에 설치된 [Google Cloud SDK Shell] 실행하면 아래 내용을 물어봅니다.

```sh
1. Choose Account : 1 (자신의 google id 선택)
2. Pic cloud project : 2 (자신의 Project 선택, GCP > Home > Dashboard > Project info > Project ID 에서 확인 가능)
3. Configure Region/Zone : Y, 46 (자신의 zone 선택, asia-east2-c 기준 46)
```

<br/>


### 3-2-2) gcloud 업그레이드 및 kubectl 설치

```sh
gcloud components update
```

```sh
gcloud components install kubectl
```

<br/>
GCP > Kubernetes Engine > Clusters > k8s-cluster 리스트에 [Connect] 버튼 클릭하여 나오는 팝업에서 아래 내용 복사
<br/>

`gcloud container clusters get-credentials k8s-cluster --zone asia-east2-c --project turnkey-conduit-258023`

<br/>

Local GCP Shell에 붙여 놓기를 한 후 아래 Node 조회 명령어로 

```sh
kubectl get nodes
```



</p>
</details>

<br/>
<br/>


![install-13](./images/install-13.jpg)

## 4-1) Dashboard

<details><summary>show</summary>
<p>

### 4-1-1) Dashboard 설치 

해당 설정은 교육목적으로 권한 설정을 모두 해제하는 방법이기 때문에 프로젝트에서 사용하실때는 이점 유의바래요
<br/>
>https://github.com/kubernetes/dashboard


```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```


### 4-1-2) 권한 해지 설정 

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

Dashboard의 Admin권한 부여 파일 만들기

```sh
echo apiVersion: rbac.authorization.k8s.io/v1beta1>> d-role.yaml&& echo kind: ClusterRoleBinding>> d-role.yaml&& echo metadata:>> d-role.yaml&& echo   name: kubernetes-dashboard>> d-role.yaml&& echo   labels:>> d-role.yaml&& echo     k8s-app: kubernetes-dashboard>> d-role.yaml&& echo roleRef:>> d-role.yaml&& echo   apiGroup: rbac.authorization.k8s.io>> d-role.yaml&& echo   kind: ClusterRole>> d-role.yaml&& echo   name: cluster-admin>> d-role.yaml&& echo subjects:>> d-role.yaml&& echo - kind: ServiceAccount>> d-role.yaml&& echo   name: kubernetes-dashboard>> d-role.yaml&& echo   namespace: kube-system>> d-role.yaml
```

적용하기
```sh
kubectl apply -f d-role.yaml
```

### 4-1-3) proxy 띄우기	

```sh
kubectl proxy
```

### 4-1-4) 접속 URL 

```sh
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.
```


</p>
</details>


