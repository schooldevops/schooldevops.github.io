---
layout: post
title:  "Kubernetes ConfigMap 실습  "
date:   2021-03-23 11:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, ConfigMap]
toc: true
---

# ConfigMap 실습

## POD 환경변수 살펴보기. 

```go
kubectl describe pod test-pod

... 생략
Containers:
  webapp-color:
    Container ID:   docker://58f186937c77a6a32da6e830b214d7f25850bfaeab8e91aa952c43cb26cb2cf4
    Image:          kodekloud/webapp-color
    Image ID:       docker-pullable://webapp-color@sha256:99c3821651452e7675af243485034a72eb1423
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 23 Mar 2021 07:01:03 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      APP_COLOR:  pink
...
```

- Environment: 부분이 env 을 지정한 부분이다. 
- APP_COLOR을 키로 하고 pink 가 값이 된다. 


## manifest 변경하기 

Env 부분을 변경해보자. 

```go
kubectl edit pod test-pod

... 생략 
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: green
...
```

spec.containers 하위에 존재하는 env 부분을 위와 같이 변경하면 설정값을 변경할 수 있다. 

## ConfigMap 조회하기. 

```go
kubectl get configmap
or
kubectl get cm

NAME        DATA   AGE
db-config   3      37s
```

## ConfigMap 상세정보 조회하기. 

```go
kubectl describe configmap db-config
Name:         db-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DB_HOST:
----
SQL01.example.com
DB_NAME:
----
SQL01
DB_PORT:
----
3306
Events:  <none>
```

위와 같이 ConfigMap 이름이 db-config 이고, 해당 ConfigMap 내에 존재하는 키와 값을 확인할 수 있다. 

## 명령적 방법으로 ConfigMap 생성하기. 

```go
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue

configmap/webapp-config-map created
```

```go
kubectl get configmaps

NAME                DATA   AGE
db-config           3      5m29s
webapp-config-map   1      29s
```

## ConfigMap manifest 파일 생성하기. 

```go
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue --dry-run=client -o yaml > config-map.yaml

cat config-map.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: webapp-config-map
data:
  APP_COLOR: darkblue
```

## 기존 pod 에 config-map 설정하기. 

### manifest 파일 뽑아내기. 

```go
kubectl get pod webapp-color -o yaml > new-pod.yaml

```

### manifest 파일 수정하기. 

```go
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color 
spec:
  containers:
    - name: webapp-color
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: webapp-config-map
              key: APP_COLOR
  restartPolicy: Never
```
