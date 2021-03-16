---
layout: post
title:  "Kubernetes Resource Limit 실습  "
date:   2021-03-16 08:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, Resource, Limit, Request]
toc: true
---


# Resource Limit 실습

## Resource CPU

- 1(1000m) : CPU 1개를 fully 사용하는것 
- 250m: CPU 1개를 1/4 만큼 사용하는것

## Resource Memory

- 1G (Gigabyte) = 1,000,000,000 bytes
- 1M (Megabyte) = 1,000,000 bytes
- 1K (Kilobyte) = 1,000 bytes

- 1Gi (Gibibyte) = 1,073,741,824 bytes
- 1Mi (Mebibyte) = 1,048,576 bytes
- 1Ki (Kibibyte) = 1,024 bytes


## Resource 설정관련

- CPU 의 경우 Limit 를 일시적으로 넘어갈 수 있음, POD 의 실행에 큰 영향을 주지 않음
- Memory 의 경우 Limit 를 넘어서면 OOMKilled 라는 메시지와 함께 POD 가 Kill 됨, 그러므로 Memory 의 Limit 을 재조정 해야할 필요가 있음 

예)

```go
kubectl get pods

NAME       READY   STATUS      RESTARTS   AGE
test-pod   0/1     OOMKilled   2          26s
```
## Resource 설정 

```go
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

resource 의 경우 위와 같이 containers 하위에 resources 항목에 지정을 한다. 

- app 컨테이너
  - 요청: 
    - 메모리: 64 Mibibyte 로 설정
    - CPU: 250m 이므로 1개의 CPU를 1/4로 사용
  - Limit:
    - 메모리: 128 Mibibyte 로 설정
    - CPU: 500m 이므로 1개의 CPU를 1/2로 사용 

