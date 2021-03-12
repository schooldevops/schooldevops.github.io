---
layout: post
title:  "Kubernetes Deployment 실습"
date:   2021-03-12 10:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, Deployment]
toc: true
---

# Deployment 살펴보기

## Deployment 확인하기. 

```go
kubectl get deploy
or
kubectl get deployment

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   0/4     4            0           7s
```

## Deoloyment 상세보기

```go
kubectl describe deploy frontend-deployment 

Name:                   frontend-deployment
Namespace:              default
CreationTimestamp:      Fri, 12 Mar 2021 00:41:31 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=busybox-pod
Replicas:               4 desired | 4 updated | 4 total | 0 available | 4 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=busybox-pod
  Containers:
   busybox-container:
    Image:      busybox888
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      echo Hello Kubernetes! && sleep 3600
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   frontend-deployment-56d8ff5458 (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m31s  deployment-controller  Scaled up replica set frontend-deployment-56d8ff5458 to 4
```

## Deployment 매니페스트 생성하기.

my-deployment.yml

```go
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox
  template:
    metadata:
      labels:
        name: busybox
    spec:
      containers:
      - name: busybox-container
        image: busybox
```

### Deployment 매니페스트 실행하기. 

```go
kubectl create -f my-deployment.yaml 
deployment.apps/deployment-1 created
```

## Commandline 으로 Deployment 생성하기. 

```go
kubectl create deployment httpd-frontend --replicas=3 --image=httpd:2.4-alpine --dry-run=client -o yaml > my-deployment2.yaml
```

```go
cat my-deployment2.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: httpd-frontend
  name: httpd-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd-frontend
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: httpd-frontend
    spec:
      containers:
      - image: httpd:2.4-alpine
        name: httpd
        resources: {}
status: {}
```

## 기본적인 커맨드 (명령적 방법)

안정적인 서비스 운영을 위해서는 명령적 방법은 가능하면 사용하지 않고, yml 파일을 통한 선언적 방법으로 이용하기. 

아래 내용은 참고차~. 

### POD Object 생성. 

```go
kubectl run --image=nginx nginx
```

### 디플로이 생성 

```go
kubectl create deployment --image=nginx nginx
```

### Service 생성. 

```go
kubectl expose deployment nginx --port 80
```

### Deployment 수정

```go
kubectl edit deployment nginx
```

### Scaling

```go
kubectl scale deployment nginx --replicas=5
```

### 이미지 갱신 

```go
kubectl set image deployment nginx nginx=nginx:1.18
```


