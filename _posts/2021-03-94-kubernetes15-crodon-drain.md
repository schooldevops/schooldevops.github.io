---
layout: post
title:  "Kubernetes Cordon, Drain 실습  "
date:   2021-03-24 15:01:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, Cordon, Drain]
toc: true
---

# Node Cordon, Drain 

## Node 살펴보기. 

```go
kubectl get nodes
```

## pod 목록 보기. 

```go
kubectl get pods -o wide
```

## Cordon

cordon 은 특정 node 에 스케줄 되지 않도록 한다. 

```go
kubectl cordon node01 
```

## Drain

drain 은 cordon 처럼 pod가 특정 노드로 스케줄 되지 않도록 하며, 또한 해당 노드에서 수행중인 pod 를 다른 pod로 이주 시키는 역할을 한다. 

```go
kubectl drain node01 --ignore-daemonsets

```

위와 같이 수행되는 경우 ReplicaSet 이나 ReplicaController 에 의해서 관리되는 pod 는 자동으로 다른 node로 스케줄링이 수행이 된다. 

그러나 단일 pod 로 수행중인 pod 가 존재하면 해당 node 는 drain 을 수행할 수 없다는 경고가 나온다. 

```go
kubectl drain node01 --ignore-daemonsets

....생략
error: cannot delete Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet (use to override): default/test-pod

```

위의 경우에는 강제로 drain 을 수행할 수 있다. 

```go
kubectl drain node01 --ignore-daemonsets --force
```

위와 같이 하면 강제로 해당 pod 를 해당 노드에서 제거한다. 

중요한 것은 drain 으로 인해서 pod 가 제거되면 다시 생성이 되지 않는다는 점이다. 

## node 다시 스케줄 되도록 하기. 

```go
kubectl uncordon node01

```

위와 같이 수행하면 node01 로 다시 스케줄이 진행이 된다. 
