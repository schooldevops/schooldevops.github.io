---
layout: post
title:  "Kubernetes 명령적 방법 및 선언전 방법 실습"
date:   2021-03-12 11:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, Imperative, Declative]
toc: true
---

# 명령형 및 선언적 사용  

## POD 생성하기. 

```go
kubectl run nginx-pod --image=nginx:alpine

pod/nginx-pod created
```

```go
kubectl get pods

NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          92s
```

###  매니페스트 생성하기. 

```go
kubectl run redis --image=redis:alpine --dry-run=client -o yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis:alpine
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### 파일 생성하기. 

```go
kubectl run redis --image=redis:alpine --dry-run=client -o yaml > redis-pod.yaml
```

다음과 같이 수정하여 레이블정보를 추가해보자. 

```go
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
    tier: db
  name: redis
spec:
  containers:
  - image: redis:alpine
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- labels.tier 을 db 로 변경해보자. 

### 적용하기. 

```go
kubectl apply -f redis-pod.yaml

kubectl get pods

NAME        READY   STATUS    RESTARTS   AGE
redis       1/1     Running   0          8s
```

정상으로 생성되는 것을 확인할 수 있다. 

### POD 를 Port와 함께 생성하기. 

```go
kubectl run user-nginx --image=nginx --port=8080

pod/user-nginx created
```

### 생성된 pod의 상세 정보 확인하기. 

```go
kubectl describe pod user-nginx
Name:         user-nginx
Namespace:    default
Priority:     0
Node:         node01/172.17.0.32
Start Time:   Fri, 12 Mar 2021 02:07:22 +0000
Labels:       run=user-nginx
Annotations:  <none>
Status:       Running
IP:           10.244.1.7
IPs:
  IP:  10.244.1.7
Containers:
  user-nginx:
    Container ID:   docker://6fa96a80b87fea1647a73cd177e5ac5859a0d5b7bc2c96cc9b9145d212cf0578
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:d5b6b094a614448aa0c48498936f25073dc270e12f5fcad5dc11e7f053e73026
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 12 Mar 2021 02:07:30 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-79wsr (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-79wsr:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-79wsr
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  88s   default-scheduler  Successfully assigned default/user-nginx to node01
  Normal  Pulling    87s   kubelet, node01    Pulling image "nginx"
  Normal  Pulled     81s   kubelet, node01    Successfully pulled image "nginx" in 5.976727266s
  Normal  Created    81s   kubelet, node01    Created container user-nginx
  Normal  Started    80s   kubelet, node01    Started container user-nginx
```

- Port: 8080/TCP 으로 생성된 것을 확인할 수 있다. 
  

## 서비스 생성하기. 

```go
kubectl expose pods redis --port=6379 --name=redis-service

service/redis-service exposed
```

### 서비스 살펴보기. 

```go
kubectl get svc -o wide

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE   SELECTOR
redis-service   ClusterIP   10.109.30.183   <none>        6379/TCP   31s   run=redis,tier=db
```

## Deployment 생성하기. 

```go
kubectl create deployment webapp --image=my-web --replicas=2

deployment.apps/webapp created
```

### 생성된 Deployment 확인하기. 

```go
kubectl get deploy -o wide
NAME     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS     IMAGES                   SELECTOR
webapp   2/2     2            2           50s   my-web         my-web                   app=webapp
```

## Namespace 생성하기

```go
kubectl create namespace my-namespace

namespace/my-namespace created
```

### 상세 보기. 

```go
kubectl get namespace

NAME              STATUS   AGE
default           Active   19m
my-namespace      Active   41s
kube-system       Active   19m
```

## Namespace 에 Deployment 생성하기. 

```go
kubectl create deployment redis-deploy --image=redis -n my-namespace

deployment.apps/redis-deploy created
```

### 생성된 객체 살펴보기. 

```go
kubectl describe deploy redis-deploy

Error from server (NotFound): deployments.apps "redis-deploy" not found
```

기본 Namespace 에 해당 deploy가 존재하지 않음 

```go
kubectl describe deploy redis-deploy -n my-namespace
Name:                   redis-deploy
Namespace:              my-namespace
CreationTimestamp:      Fri, 12 Mar 2021 02:12:23 +0000
Labels:                 app=redis-deploy
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=redis-deploy
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=redis-deploy
  Containers:
   redis:
    Image:        redis
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   redis-deploy-68fb445555 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------  
  Normal  ScalingReplicaSet  95s   deployment-controller  Scaled up replica set redis-deploy-68fb445555 to 1

```

namespace 를 명확히 지정하여 정상으로 확인 가능하다. 

## 매니페스트 POD 를 만들고, Service 연결하기. 

### POD 매니페스트 생성하기. 

```go
kubectl run httpd --image=httpd:alpine --dry-run=client -o yaml > httpd-pod.yaml
```

### pod 적용하기. 

```go
kubectl create -f httpd-pod.yaml 

pod/httpd created
```

### Service 연동하기. 

```go
kubectl expose pods httpd --type=ClusterIP --port=80 --dry-run=client -o yaml > httpd-service.yaml
```

### 적용하기. 

```go
kubectl create -f httpd-service.yaml 

service/httpd created
```