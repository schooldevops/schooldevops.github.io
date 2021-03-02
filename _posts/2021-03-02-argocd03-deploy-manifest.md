---
layout: post
title:  "Kubernetes ArgoCD 디플로이 "
date:   2021-03-02 15:58:49 +0900
categories: Kubernetes
tags: [kubernetes, argocd, ci, cd, gitops, deploy, docker, dockerhub]
toc: true
---

# ArgoCD 디플로이 

이제는 ArgoCD 를 이용하여 kubernetes 에 배포를 수행할 것이다. 

그러기 위해서는 기본적으로 2가지 작업을 수행해 주어야한다. 

- Deployment: Kubernetes 에 어플리케이션을 배포하기 위해서는 Deployment 를 통해서 배포를 수행하게 된다. 버전관리 등 다양한 이점이 있다. 
- Service: Service 는 외부 접속을 위한 연결 정의를 수행하는 매니페스트이다. 

## Deployment 작성하기. 

greetweb-deploy.yml 파일을 다음과 같이 작성한다. 

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: greet
  name: greet 
spec:
  replicas: 2
  selector:
    matchLabels:
      app: greet
  template:
    metadata:
      labels:
        app: greet 
    spec:
      containers:
        - image:  unclebae/gogreet:v1.0
          name:  unclebae-gogreet
```

## Service 작성하기 

서비스를 생성한다. 

```yml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: greet
  name: greet
  namespace: default 
spec:
  ports:
  - port: 9999
    protocol: TCP
    targetPort: 8080
  selector:
    app: greet
  type: NodePort

```

## 위 작성한 코드를 모두 github으로 올린다. 

## ArgoCD 접속하기. 

https://localhost:31602/

으로 지난번에 argocd 를 설치 했었다. 이제는 argocd 로 배포를 수행해 보자. 

### 리포지토리 등록하기 

우선 argocd 레 우리가 작성한 디플로이먼트 를 수행하기 위해서는 다음과 같은 리포지토리 등록 절차가 필요하다.

![argocd_deploy_01](https://user-images.githubusercontent.com/66154381/109616508-902b9780-7b78-11eb-900d-b32873ba105f.png)

- Manage your repositories, projects, settings 탭을 선택하다.
- Repositories 부분을 클릭해준다. 

![argocd_deploy_02](https://user-images.githubusercontent.com/66154381/109616568-a5a0c180-7b78-11eb-9391-e5bf63603825.png)

- 위 화면에서 CONNECT REPO USING HTTPS 를 선택하자. 
  
![argocd_deploy_03](https://user-images.githubusercontent.com/66154381/109616606-b3eedd80-7b78-11eb-9974-9dd83d92ca59.png)

- Type: git
- Repository URL: 배포 매니페스트가 존재하는 리포지토리를 추가한다.
  - https://github.com/schooldevops/argocd-tutorials 을 입력해준다. 
- Username / Password: 위 리포지토리는 오픈이기 때문에 입력없이 스킵한다. (private repository라면 username/password가 필요하다.)
- TLS 부분은 그냥 넘어가자. 
- Skip Server verification: 기본으로 유지한다. 
- Enable LFS support: 기본으로 유지한다. 

생성하기를 완료하면 리포지토리 목록을 확인할 수 있게 된다. 

![argocd_deploy_04](https://user-images.githubusercontent.com/66154381/109616646-c406bd00-7b78-11eb-8a21-63b487d1d4ea.png)

### Application 생성하기. 

이제는 Kubernetes 에 배포해보자. 

왼쪽 탭 맨 상단의 탭을 클릭한다. 

![argocd_deploy_05](https://user-images.githubusercontent.com/66154381/109616705-d7198d00-7b78-11eb-83eb-e2aff7b6c47c.png)

CREATE APPLICATION 을 선택하자. 

![argocd_deploy_06](https://user-images.githubusercontent.com/66154381/109616757-e567a900-7b78-11eb-8dcc-e8fbcb44dfe8.png)

- Application Name: greet 로 지정했다. 
- Project: default 로 그대로 두자. 
- SYNC POLICY: 
  - Manual: git과 kubernetes 의 배포 상태가 다른경우 수동으로 싱크할경우 사용한다. 
  - Auto: 자동으로 git 코드가 변경이 되면 kubernetes에 반영하게 된다. 
- USE A SCHEMA TO VALIDATE RESOURCE MANIFESTS: 기본 체크로 그대로 둔다. (스키마 검사를 수해할지 여부이다.)
- AUTO-CREATE NAMESPACE: 매니페스트의 네임스페이스가 없는경우 자동 생성할지 여부를 설정한다. (기본으로 언체크한다.)
- Repository URL: 등록한 리포지토리에서 선택한다. 
- Revision: HEAD는 최신의 리비젼을 사용하도록 한다. 
- Path: 매니페스트 위치가 deploy 이므로 우리는 deploy로 잡았다. 


![argocd_deploy_07](https://user-images.githubusercontent.com/66154381/109616805-f4e6f200-7b78-11eb-9e36-54f81f2b610a.png)

- Cluster URL: 현재 context에 있는 cluster 의 정보를 지정한다. ArgoCD가 kubernetes 에서 올라가 있으므로, 현재 kubernetes의 엔드포인트가 된다.
- Namespace: 별도 네임스페이스를 작성하기 않았으므로, default 를 기술한다. 
- DIRECTORY RECURSE: 디렉토리가 계층적이라면 체크한다. 아닌경우 그냥 언체크로 둔다. 
- 이후 부분은 아규먼트와 환경값이 없으므로 그대로 둔다. 

생성하기를 클릭하면 다음과 같은 내용을 볼 수 있다. 

![argocd_deploy_08](https://user-images.githubusercontent.com/66154381/109616847-03cda480-7b79-11eb-9c53-a338f54bc1cf.png)

- 현재 상태는 OutOfSync로 주황색 표시가 되어 있음을 알 수 있다. 

볼드된 어플리케이션을 클릭하여 상세 내용에 들어가 보자. 

![argocd_deploy_09](https://user-images.githubusercontent.com/66154381/109616880-11832a00-7b79-11eb-8d92-60308079dd95.png)

위 그림과 같이 greet 어플리케이션 형상을 보여준다. 

- greet service 가 존재하며 서비스 연결을 위한 설정정보가 있다. 
- greet deploy 는 배포를 수행할 deployment 매니페스트이다. 

아직 싱크가 되지 않았으며, 배포가 이루어 지지 않았음을 알 수 있다. 

### App Detail 속성 보기

좌측 App Details 버튼을 클릭하면 앱 상세 정보를 확인할 수 있다. 

![argocd_deploy_10](https://user-images.githubusercontent.com/66154381/109616927-2069dc80-7b79-11eb-9b09-06c82308ea50.png)

우리가 작성한 그대로의 정보를 확인할 수 있다. 

### 동기화 하기 

이제 동기화를 통해서 Kubernetes 에 우리의 어플리케이션을 배포해보자. 

Sync 를 클릭한다. 그럼 우측에 패널이 나타나며, 몇가지 옵션들을 선택할 수 있다. 

![argocd_deploy_11](https://user-images.githubusercontent.com/66154381/109617003-2fe92580-7b79-11eb-930d-f8481d7a02b1.png)

- Synchronizing ... from: 싱크될 매니페스트 위치를 알려준다. 
- Revision: 배포할 리비젼 정보를 보여준다. 여기서는 HEAD 최신 내용임을 알 수 있다. 
  - Kubernetes 에서 Deployment 시에 revision 옵션이 자동으로 설정되므로, 배포 히스토리를 알 수 있다. 
  - 이 속성을 통해서 롤백도 가능하다. 
- 아래 정보들은 배포한 디플로이먼트를 정리, 테스트로 수행해보기 등 다양한 옵션들이 있다. 이 부분은 차후 상세히 알아보겠다. 

SYNCHRONIZE 를 클릭하여 실제 Kubernetes 에 반영한다. 

### 반영된 형상 확인하기. 

이제 실제 반영된 형상을 살펴 보자. 

![argocd_deploy_12](https://user-images.githubusercontent.com/66154381/109617044-3b3c5100-7b79-11eb-966e-606a774c43b8.png)

초기 준비 단계에서 sync 된 이후의 상태를 보면 다음과 같은 내용을 확인할 수 있다. 

- Service: 서비스 엔드포인트가 배포 되었음을 확인할 수 있다. 
  - EP: 실제 서비스를 구현하는 여러 엔드포인트들의 묶음을 의미한다. 
  - endpointslice: 서비스를 구현하는 엔드포인트의 하위 집합을 말한다. 실제 엔드포인트 구현체이다.
- Deployment: 
  - RS: 한개의 리플리카셋을 확인할 수 있다. 리플리카 셋이 Pod의 리플리케이션을 컨트롤한다. 
    - 2개의 Pod: 리플리카 셋에 정의한 리플라카 값이 2 이므로 2개의 Pod가 생성되었다. 


## 배포된 서비스 확인하기. 

```go
kubectl get svc

NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
greet        NodePort    10.102.28.18   <none>        9999:32496/TCP   44s
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          8d
```

## 접속 테스트하기. 

```go
curl localhost:32496/health
OK

curl localhost:32496/hello/schooldevops
Hello schooldevops
```

정상으로 완료된것을 확인되었다. 

## 결론

지금까지 웹 프로그램을 생성하고, docker hub에 추가하였으며, docker hub의 이미지를 가져와서 로컬 쿠버네티스에 argoCD를 이용하여 배포 해 보았다. 

몇가지 단계를 거쳐서 소위 말하는 GitOps를 구현해 보았고, 어떻게 ArgoCD를 이용하는지 퀵하게 알아 보았다. 

ArgoCD는 시각적으로 투명하게 배포 작업을 수행할 수 있도록 제공해주고, 간단한 설정을 통해서 배포 자동화를 구현할 수 있는 괞찮은 CD틀인것 같다. 
