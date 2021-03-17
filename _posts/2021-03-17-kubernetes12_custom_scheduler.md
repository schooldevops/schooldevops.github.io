---
layout: post
title:  "Kubernetes Custom Scheduler 실습  "
date:   2021-03-17 11:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, CustomScheduler]
toc: true
---

# CustomScheduler 실습

## 기본 스케줄러 확인하기. 

```go
kubectl get pods -n kube-system | grep -i scheduler

kube-scheduler-controlplane            1/1     Running   0          4m53s
```

## 스케줄러 정보 상세 보기. 

```go
kubectl describe pods -n kube-system kube-scheduler-controlplane

Name:                 kube-scheduler-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 controlplane/172.17.0.11
Start Time:           Wed, 17 Mar 2021 11:40:56 +0000
Labels:               component=kube-scheduler
                      tier=control-plane
Annotations:          kubernetes.io/config.hash: 5146743ebb284c11f03dc85146799d8b
                      kubernetes.io/config.mirror: 5146743ebb284c11f03dc85146799d8b
                      kubernetes.io/config.seen: 2021-03-17T11:40:49.302668789Z
                      kubernetes.io/config.source: file
Status:               Running
IP:                   172.17.0.11
IPs:
  IP:           172.17.0.11
Controlled By:  Node/controlplane
Containers:
  kube-scheduler:
    Container ID:  docker://7f9ab656138586b9758983e0b733cb583e9dcd194ba5ed44ebeeb7e728f6bfe5
    Image:         k8s.gcr.io/kube-scheduler:v1.19.0
    Image ID:      docker-pullable://k8s.gcr.io/kube-scheduler@sha256:529a1566960a5b3024f2c94128e1cbd882ca1804f222ec5de99b25567858ecb9
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-scheduler
      --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
      --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
      --bind-address=127.0.0.1
      --kubeconfig=/etc/kubernetes/scheduler.conf
      --leader-elect=true
      --port=0
    State:          Running
      Started:      Wed, 17 Mar 2021 11:40:35 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Liveness:     http-get https://127.0.0.1:10259/healthz delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get https://127.0.0.1:10259/healthz delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/kubernetes/scheduler.conf from kubeconfig (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kubeconfig:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/scheduler.conf
    HostPathType:  FileOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecuteop=Exists
Events:            <none>
```

## Scheduler 파일 위치 

```go
ls -alt /etc/kubernetes/manifests/

total 24
drwxr-xr-x 4 root root 4096 Mar 17 11:40 ..
drwx------ 2 root root 4096 Mar 17 11:40 .
-rw------- 1 root root 2096 Mar 17 11:40 etcd.yaml
-rw------- 1 root root 3663 Mar 17 11:40 kube-apiserver.yaml
-rw------- 1 root root 3345 Mar 17 11:40 kube-controller-manager.yaml
-rw------- 1 root root 1384 Mar 17 11:40 kube-scheduler.yaml
```

커스텀 스케줄러를 만들고자 한다면 해당 스케줄러 매니페스트를 복사하여 새로운 스케줄러를 생성한다. 

## pod 를 특정 스케줄러로 실행하기. 

```go
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
     name: nginx
  schedulerName: customer-scheduler
```

schedulerName: 을 지정하여 특정 스케줄러로 실행하기. 

## pod 살펴보기. 

```go
watch "kubectl get pods | grep nginx"
```
