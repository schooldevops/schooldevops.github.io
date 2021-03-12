---
layout: post
title:  "Kubernetes ReplicaSet 실습"
date:   2021-03-11 11:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, ReplicaSet]
toc: true
---

# ReplicaSet 살펴보기

## 리플리카 셋이 없는경우 

```go
kubectl get rs

No resources found in default namespace.

```

## 리플리카 셋이 있는경우 

```go
kubectl get rs

NAME              DESIRED   CURRENT   READY   AGE
replica-set   4         4         0       25s
```

- DESIRED: 원하는 POD의 개수
- CURRENT: 현재 존재하는 POD의 개수 
- READY: 운영 상태가 된 POD수 (위 상태를 보면 현재 요청한 POD와 현재 POD는 4개이나, 정상적으로 서비스 되는 것은 0개이다.)

## 리플리카 셋 상세보기. 

```go
kubectl get rs -o wide

NAME              DESIRED   CURRENT   READY   AGE    CONTAINERS          IMAGES       SELECTOR
replica-set   4         4         0       117s   busybox-container   busybox123   name=busybox-pod
```

위 내용을 확이해보면 busybox 컨테이에 대한 ReplicaSet 을 확인할 수 있으며, 

이미지는 busybox123 이다. 아직 정상으로 수행되지 않았음을 확인할 수 있다. 

## 리플리카셋 내용 상세보기

```go
kubectl describe rs replica-set
Name:         replica-set
Namespace:    default
Selector:     name=busybox-pod
Labels:       <none>
Annotations:  <none>
Replicas:     4 current / 4 desired
Pods Status:  0 Running / 4 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  name=busybox-pod
  Containers:
   busybox-container:
    Image:      busybox777
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      echo Hello Kubernetes! && sleep 3600
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  42s   replicaset-controller  Created pod: replica-set-l7zdk
  Normal  SuccessfulCreate  42s   replicaset-controller  Created pod: replica-set-nk2xw
  Normal  SuccessfulCreate  42s   replicaset-controller  Created pod: replica-set-drthq
  Normal  SuccessfulCreate  42s   replicaset-controller  Created pod: replica-set-bb8sf
```

## 현재 POD 살펴보기 

```go
kubectl get pods

NAME                    READY   STATUS             RESTARTS   AGE
replica-set-gvc2n   0/1     ImagePullBackOff   0          2m8s
replica-set-jxrnk   0/1     ImagePullBackOff   0          2m8s
replica-set-v2fhj   0/1     ImagePullBackOff   0          2m8s
replica-set-w6jrj   0/1     ImagePullBackOff   0          2m8s
```

보는 바와 같이 4개의 POD가 실행이 되었으나 ImagePullBackOff 즉, 이미지를 다운받을 수 없다는 이유료 인해서 정상으로 수행되지 않고 있다. 

ReplicaSet 내에서 POD 삭제하기. 

```go
kubectl delete pod replica-set-gvc2n

pod "replica-set-gvc2n" deleted

```

### 확인하기. 

```go
kubectl get pods

NAME                    READY   STATUS             RESTARTS   AGE
replica-set-4ltpk   0/1     ImagePullBackOff   0          18s
replica-set-jxrnk   0/1     ImagePullBackOff   0          6m25s
replica-set-v2fhj   0/1     ErrImagePull       0          6m25s
replica-set-w6jrj   0/1     ImagePullBackOff   0          6m25s
```

위 내용과 같이 'replica-set-gvc2n' 은 삭제 되었고 신규로 'replica-set-4ltpk' 이 새로 생성 되었음을 확인 할 수 있음

ReplciaSet 은 POD 를 Desired 에서 정의한 개수만큼 유지하기 위해서 노력한다. 

리플리카 셋 실패 케이스 1, 올바른 버젼 정보가 존재하지 않는경우 .

```go
kubectl apply -f replicaset-desc.yaml 

error: unable to recognize "replicaset-desc.yaml": no matches for kind "ReplicaSet" in version "v1"
```

## ReplicaSet 만들기. 

```go
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

## ReplicaSet 정상 실행 케이스 

```go
kubectl apply -f replicaset-nginx.yaml 

replicaset/replicaset created
```

### 현재 상태 확인하기. 

```go
kubectl get rs -o wide

NAME              DESIRED   CURRENT   READY   AGE   CONTAINERS          IMAGES       SELECTOR
replicaset-nginx      2         2         2       49s   nginx               nginx        tier=frontend
```

replicaset-nginx 의 경우 다음과 같은 내용을 확인할 수 있다. 

- NAME: 리플리카 셋의 이름
- DESIRED: 요청 개수 (2개)
- CURRENT: 현재 pod 의 개수
- READY: 요청건
- AGE: 처음 실행 시간부터 현재까지 초
- CONTAINER: 컨테이너 이름
- IAMGES: nginx (이밎를 풀 가져온 이미지)
- SELECTOR: 셀렉터 (사욘자에 편의 제공)

## 내역 수정 번역하기. 

```go
kubectl apply -f replicaset-defsc.yaml 

replicaset/replicaset created
```

## ReplicaSet 삭제하기. (복수개)

```go
kubectl delete rs replicaset-1 replicaset-2  

replicaset.apps "replicaset-1" deleted
replicaset.apps "replicaset-2" deleted
```

### 개별 POD 제거해보기. 

```go 
kubectl delete pods replica-set-v2fhj   

pod "replica-set-v2fhj" deleted
```

## ReplicaSet 오류 수정 및 재기동

```go
kubectl edit rs replica-set

-- 리플리카 셋은 변경되었으나 pods 는 변경이 안되었음. 그러므로 모두 삭제 해주거나 스케일 변경이 필요. 

kubectl get pods
NAME                    READY   STATUS             RESTARTS   AGE
replica-set-drthq   0/1     ImagePullBackOff   0          15m
replica-set-kf22t   0/1     ImagePullBackOff   0          11m
replica-set-l7zdk   0/1     ImagePullBackOff   0          15m
replica-set-nk2xw   0/1     ImagePullBackOff   0          15m

```

### 모두 삭제 방법

```go
kubectl delete pods replica-set-drthq replica-set-kf22t replica-set-l7zdk replica-set-nk2xw
```

### 스케일 변경 방법 

```go
kubectl scale rs new-replica-set --replicas=0
replicaset/replica-set scaled

kubectl scale rs new-replica-set --replicas=4
replicaset/replica-set scaled
```

