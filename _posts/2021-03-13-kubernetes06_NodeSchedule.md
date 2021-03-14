---
layout: post
title:  "Kubernetes Nodel 선택하기 "
date:   2021-03-13 11:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, Imperative, Declative, Selector, nodeSelect]
toc: true
---


# Node 선택하기

Node를 선택하여 POD 배포하기. 

```go
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  nodeName: dev-node01
status: {}
```

위 내역은 nodeName: dev-node01 를 통해서 dev-node01 에 위 pod 를 생성한다. 


## Selector 로 실행중인 노드 검색하기. 

### env=dev 라는 레이블된 노드 찾기. 

```go
kubectl get pods --selector env=dev
or
kubectl get pods -l env=dev

NAME          READY   STATUS    RESTARTS   AGE
app-1-hjf6l   1/1     Running   0          73s
app-1-j8j6t   1/1     Running   0          73s
app-1-xt5rj   1/1     Running   0          73s
db-1-2rwp6    1/1     Running   0          73s
db-1-5nj26    1/1     Running   0          73s
db-1-t4xjs    1/1     Running   0          73s
db-1-vd744    1/1     Running   0          73s
```

## pod 갯수찾기. 

```go
kubectl get pods -l env=dev --no-headers | wc -l
```

## env=prod 인 전체 Object 조회

```go
kubectl get all --selector env=prod

NAME              READY   STATUS    RESTARTS   AGE
pod/app-1-zzxdf   1/1     Running   0          4m17s
pod/app-2-r67m2   1/1     Running   0          4m18s
pod/auth          1/1     Running   0          4m17s
pod/db-2-2zgtl    1/1     Running   0          4m17s

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/app-1   ClusterIP   10.102.76.183   <none>        3306/TCP   4m17s

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/app-2   1         1         1       4m18s
replicaset.apps/db-2    1         1         1       4m17s
```

## 복수개의 레이블된 pod 객체 조회 

```go
kubectl get pod --selector env=prod,team=settle,tier=backend

NAME          READY   STATUS    RESTARTS   AGE
app-1-zzxdf   1/1     Running   0          7m24s
```
