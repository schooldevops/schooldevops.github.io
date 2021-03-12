---
layout: post
title:  "Kubernetes Service 실습"
date:   2021-03-12 11:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, Service]
toc: true
---

# Service 살펴보기

## Service 확인하기. 

```go
kubectl get svc

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m57s
```

## 서비스 상세 보기. 

```go
kubectl describe svc kubernetes

Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         172.17.0.15:6443
Session Affinity:  None
Events:            <none>
```

- 기본 네임스페이스 default 를 사용한다. 
- Type: ClusterIP 이므로 클러스터 내부에서 접근할 수 있다. 
- Port: pod 내 연결된 포트
- TargetPort: 노출되는 포트 

## 서비스 매니페스트 생성하기. 

```go
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 8080
      nodePort: 30080
  selector:
    name: simple-webapp
```

- apiVersion: Service 는 v1이다. 
- kind: Service 임을 의미
- type: NodePort 는 클러스터 노드에서 해당 포트로 해당 서비스를 외부에 오픈해서 접근할 수 있도록 한다. 
- ports.targetPort: 서비스가 가리키는 POD 에 서비스되는 포트이다. 
- ports.port: 쿠버네티스 클러스터 내부에서 해당 포트로 접근 할 수 있도록 해준다. 
- ports.nodePort: NodePort 는 클러스터 노드에 포트가 할당된다. 외부에서 해당 노드의 IP:Port 로 접속할 수 있다. 
- selector.name: pod 의 이름이 지정된다. 

예) pod설정이 다음과 같은경우

```go
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: hello-world
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
      - containerPort: 80
```

즉 외부에서 curl nodeIP:nodePort --> targetPort --> nginx Pod 로 트래픽이 흐른다. 

그리고 클러스터 내부에서 dns:port --> targetPort --> nginx Pod 로 트래픽이 흐른다. 

