---
layout: post
title:  "Kubernetes DaemonSet 실습  "
date:   2021-03-16 11:10:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, DaemonSet]
toc: true
---

# DaemonSet 실습 

## DaemonSet 확인하기. 

```go
kubectl get daemonsets 

No resources found in default namespace.
```

## 전체 Namespace 에서 Daemonset 조회하기. 

```go
kubectl get daemonsets --all-namespaces

NAMESPACE     NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kube-flannel-ds-amd64     2         2         2       2            2           <none>                   62m
kube-system   kube-flannel-ds-arm       0         0         0       0            0           <none>                   62m
kube-system   kube-flannel-ds-arm64     0         0         0       0            0           <none>                   62m
kube-system   kube-flannel-ds-ppc64le   0         0         0       0            0           <none>                   62m
kube-system   kube-flannel-ds-s390x     0         0         0       0            0           <none>                   62m
kube-system   kube-proxy                2         2         2       2            2           kubernetes.io/os=linux   62m
```

확인하면 kubernetes network 역시 damonset 이다. 

kube-proxy 역시 daemonset 으로 확인할 수 있다. 

```go
kubectl get daemonsets --all-namespaces --no-headers | wc -l

6
```

## DaemonSet 상세보기. 

```go
kubectl describe daemonsets kube-proxy -n kube-system

Name:           kube-proxy
Selector:       k8s-app=kube-proxy
Node-Selector:  kubernetes.io/os=linux
Labels:         k8s-app=kube-proxy
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 2
Current Number of Nodes Scheduled: 2
Number of Nodes Scheduled with Up-to-date Pods: 2
Number of Nodes Scheduled with Available Pods: 2
Number of Nodes Misscheduled: 0
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           k8s-app=kube-proxy
  Service Account:  kube-proxy
  Containers:
   kube-proxy:
    Image:      k8s.gcr.io/kube-proxy:v1.19.0
    Port:       <none>
    Host Port:  <none>
    Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/config.conf
      --hostname-override=$(NODE_NAME)
    Environment:
      NODE_NAME:   (v1:spec.nodeName)
    Mounts:
      /lib/modules from lib-modules (ro)
      /run/xtables.lock from xtables-lock (rw)
      /var/lib/kube-proxy from kube-proxy (rw)
  Volumes:
   kube-proxy:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-proxy
    Optional:  false
   xtables-lock:
    Type:          HostPath (bare host directory volume)
    Path:          /run/xtables.lock
    HostPathType:  FileOrCreate
   lib-modules:
    Type:               HostPath (bare host directory volume)
    Path:               /lib/modules
    HostPathType:       
  Priority Class Name:  system-node-critical
Events:                 <none>
```

### DaemonSet node 확인하기 

```go
kubectl -n kube-system get pods -o wide | grep kube-proxy

.. pod 상세 정보 및 node 정보 확인 가능 
```

## Daemonset 생성하기. 

### deployments 우선 생성하기. 

```go
kubectl create deployment elasticsearch --image=k8s.gcr.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > es_daemonset.yaml

```

### yaml 수정하기. 

```go
apiVersion: apps/v1
kind: DaemonSet
metadata:
  creationTimestamp: null
  labels:
    app: elasticsearch
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: elasticsearch
    spec:
      containers:
      - image: k8s.gcr.io/fluentd-elasticsearch:1.20
        name: fluentd-elasticsearch
```

- kind: DaemonSet 으로 설정
- replicas: DaemonSet 에서는 의미 없으므로 제거한다. 

### 반영하기. 

```go
kubectl create -f es_daemonset.yaml 
```

```go
kubectl get ds -n kube-system

NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
elasticsearch             1         1         1       1            1           <none>                   102s
```


