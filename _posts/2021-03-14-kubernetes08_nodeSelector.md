---
layout: post
title:  "Kubernetes nodeSelector 실습  "
date:   2021-03-14 11:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, Imperative, Declative, Selector, nodeSelector]
toc: true
---


# Node 의 Label 검사하기.

```go
kubectl describe nodes node01 | grep -A10 Labels

Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
```

## Node Label 목록만 보기

```go
kubectl get nodes node01 --show-labels

```

## Node Label 추가힉. 

```go
kubectl label node node01 color=blue

node/node01 labeled

kubectl describe nodes node01 | grep -A10 Labels

Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    color=blue
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
```

## 6개의 pod 를 가진 Deployment 생성하기. 

- name: blue
- image: nginx
- replicas: 6
  
```go
kubectl create deployment blue --image=nginx --replicas=6

deployment.apps/blue created
```

## Affinity 설정하기. 

```go
cat nginx-deploy.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: red
  name: red
spec:
  replicas: 3
  selector:
    matchLabels:
      app: red
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: red
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
status: {}
```

- affinity: 이하 부분부터 노드에 맞는 Pod를 배포하게 된다. 

