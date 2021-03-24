---
layout: post
title:  "Kubernetes Secrets 실습  "
date:   2021-03-24 15:01:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, Secrets]
toc: true
---

# Secret 실습

## Secret 조회하기. 

```go
kubectl get secrets

NAME                  TYPE                                  DATA   AGE
default-token-8k111   kubernetes.io/service-account-token   3      35m
```

## Secret 상세 조회하기. 

```go
kubectl describe secrets default-token-8kx8z
Name:         default-token-8kx8z
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 637836b0-5738-491f-9886-04e4ce00b2b4

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjJxQVhXVW4zODl2R2xBZWtZYnRZWjBBNHZRN2JOZDU1X19XMDdIczF6aVEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tOGt4OHoiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjYzNzgzNmIwLTU3MzgtNDkxZi05ODg2LTA0ZTRjZTAwYjJiNCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.EWpjoUXkKTOLcbv4f4GYvHZ8IALvOZmUwS3GTl94WgZY0AVeTAWIaCc2RGSsmN1yon0SnNMpVmEoLS6hArILnzVl_0uaNNObvuPmQzHRWwOVlqZr7hdMqdMz-onuQMKfhFU4FzN9Rr9r2x3tn5VK0KL7G_ohIjNEVFcwTfqVjtZhmoyWTV1f_leuI8F7EIDv19jSQgAPyA1JKjS39qA9VNdyrKh0cga4mtkc7kIrrC99GiTRwy_SfD8XvLbBm6wme8binuGtapg9PlyDS-R_0iXTezo03iFHJCsvLLdx6DrTaD6T6L8dH5FHcRxcjC9gSj_Ro8BOnwwqGRgXArMXLQ
```

위 내용은 ca.crt, namespace, token 으로 3개의 secret 정보가 존재한다. 


## Secret 생성하기. (명령적 방법)

```go
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password

secret/db-secret created
```

### 생성 시크릿 확인하기. 

```go
kubectl get secrets

NAME                  TYPE                                  DATA   AGE
db-secret             Opaque                                3      19s
default-token-8kx8z   kubernetes.io/service-account-token   3      45m
```

### Secret Type 알아보기. 

- Opaque:	임의의 사용자 정의 데이터 (kubectl create secret generic ... 으로 이용)
- kubernetes.io/service-account-token:	서비스 어카운트 확인용 토큰
- kubernetes.io/dockercfg:	직렬화 된(serialized) ~/.dockercfg 파일
- kubernetes.io/dockerconfigjson:	직렬화 된 ~/.docker/config.json 파일
- kubernetes.io/basic-auth:	기본 인증을 위한 자격 증명(credential)
- kubernetes.io/ssh-auth:	SSH를 위한 자격 증명
- kubernetes.io/tls:	TLS 클라이언트나 서버를 위한 데이터
- bootstrap.kubernetes.io/token:	부트스트랩 토큰 데이터

[SecretType](https://kubernetes.io/ko/docs/concepts/configuration/secret/) 참조 

## Pod 에서 Secret 사용하기. 

```go
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-pod
  name: webapp-pod
  namespace: default
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: db-secret
```

envFrom 을 이용하여 secret 을 참조한다. 

## Secret 수정하기. 

```go
kubectl edit secrets db-secret

```

secret 을 수정하기 위해서는 base64 로 인코딩 된 값을 이용해야한다. 

### Encoding

```go
echo "hello" | base64

aGVsbG8K
```

### Decoding

```go
echo "aGVsbG8K" | base64 --decode

hello
```

### Secret 내용 조회하기. 

```go
kubectl get secrets/default-token-vdb9m --template={{.data.namespace}} | base64 --decode

default
```