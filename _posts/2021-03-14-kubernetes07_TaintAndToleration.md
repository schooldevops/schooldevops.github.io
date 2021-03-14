---
layout: post
title:  "Kubernetes Taint and toleration "
date:   2021-03-13 11:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, Imperative, Declative, Selector, Taint, Toleration]
toc: true
---


# Taint and Toleration

## Node 목록 보기

```go
kubectl get nodes

NAME           STATUS   ROLES    AGE    VERSION
controlplane   Ready    master   119s   v1.19.0
node01         Ready    <none>   86s    v1.19.0
```

Node 목록 상세 

```go
kubectl get nodes -o wide

NAME           STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
controlplane   Ready    master   3m1s    v1.19.0   172.17.0.8    <none>        Ubuntu 18.04.5 LTS   4.15.0-122-generic   docker://19.3.13
node01         Ready    <none>   2m28s   v1.19.0   172.17.0.16   <none>        Ubuntu 18.04.5 LTS   4.15.0-122-generic   docker://19.3.13
```

## Taint 설정하기. 

```go
kubectl taint nodes node01 spray=Value:NoSchedule

node/node01 tainted
```

kubectl taint nodes <nodeName> <key>=<value>:<effect>

의 형식으로 지정가능하다. 

## Taint Override 하기. 

```go
kubectl taint nodes node01 spray=mortein:NoSchedule --overwrite

node/node01 modified
```

--overwrite 를 이용하면 기존 taint 를 갱신할 수 있다. 

## Toleration 이 적용된 pod 생성하기. 

```go

apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
  tolerations:
  - key: spray
    value: mortein
    effect: NoSchedule
    operator: Equal

```

