---
layout: post
title:  "Kubernetes ArgoCD 설치하기"
date:   2021-02-25 20:51:49 +0900
categories: Kubernetes
tags: [kubernetes, argocd, ci, cd, gitops, deploy]
toc: true
---

# ArgoCD 사용하기. 

## 사전 확인하기. 

우선 Kubernetes 가 설치되고, kubectl 을 이용할 수 있는지 우선 확인하자. 

```shell
kubectl cluster-info

Kubernetes master is running at https://172.16.10.100:6443
KubeDNS is running at https://172.16.10.100:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

필자의 경우 로컬에 VM을 이용하여 multer master kubernetes cluster 를 우선 구현해 놓았다. 

```shell
kubectl get nodes

NAME            STATUS   ROLES    AGE    VERSION
kube-master01   Ready    master   3d8h   v1.19.2
kube-master02   Ready    master   3d8h   v1.19.2
kube-master03   Ready    master   3d8h   v1.19.2
kube-node01     Ready    <none>   3d8h   v1.19.2
kube-node02     Ready    <none>   3d8h   v1.19.2
```

마스터 3대 노드 2대로 노드 정보를 확인 할 수 있다. 

## ArgoCD 설치하기. 

일단 argocd github 사이트에 접근한다. 

[https://github.com/argoproj/argo-cd](https://github.com/argoproj/argo-cd) 이중 우측 화면에서 Release를 찾아서 클릭하자. 

![argocd01](https://user-images.githubusercontent.com/66154381/109141191-ce077500-77a0-11eb-8a2a-d2dc157a0507.png)

보는바와 같이 Release 를 클릭하자. 필자의 경우 1.8.5 버젼이 나와 있다. 

![argocd02](https://user-images.githubusercontent.com/66154381/109141350-fc855000-77a0-11eb-8d15-d289291949a0.png)

### 설치하기. 

처음 5개의 클러스터를 VM으로 설정했으나 PC가 너무 버벅여서 docker-desktop 로 설치해본다. 

```shell
kubectl create namespace argocd

namespace/argocd created
```

```shell
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v1.8.5/manifests/install.yaml

Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created

serviceaccount/argocd-application-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-redis created
serviceaccount/argocd-server created

role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-redis created
role.rbac.authorization.k8s.io/argocd-server created

clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created

rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created

clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created

configmap/argocd-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created

secret/argocd-secret created

service/argocd-dex-server created
service/argocd-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created

deployment.apps/argocd-dex-server created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created

statefulset.apps/argocd-application-controller created
```

위 내용과 같이 ArgoCD가 설치가 되었다. 

뉴라인으로 분리된 사항을 보면 ArgoCD를 위해서 어떠한 pod와 서비스가 실행이 되었고, 각 서비스 어카운트와 클러스터 롤바인딩이 있는지 이해할 수 있을 것이다. 

- argocd-server
- repo-server
- app-controller
- redis
- dex server 

등과 같은 서비스가 올라가 있다. 

### 설치된 정보 살펴보기. 

```shell
kubectl get all -n argocd

NAME                                     READY   STATUS    RESTARTS   AGE
pod/argocd-application-controller-0      1/1     Running   0          2m11s
pod/argocd-dex-server-66bc48cb59-zd7mp   1/1     Running   0          2m11s
pod/argocd-redis-6fb68d9df5-bgtpv        1/1     Running   0          2m11s
pod/argocd-repo-server-5bc4ff7b6-dsb9j   1/1     Running   0          2m11s
pod/argocd-server-68f79c4cf9-pkcv6       1/1     Running   0          2m11s

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/argocd-dex-server       ClusterIP   10.108.64.155    <none>        5556/TCP,5557/TCP,5558/TCP   2m12s
service/argocd-metrics          ClusterIP   10.105.67.231    <none>        8082/TCP                     2m12s
service/argocd-redis            ClusterIP   10.100.242.126   <none>        6379/TCP                     2m12s
service/argocd-repo-server      ClusterIP   10.110.179.92    <none>        8081/TCP,8084/TCP            2m12s
service/argocd-server           ClusterIP   10.96.1.86       <none>        80/TCP,443/TCP               2m12s
service/argocd-server-metrics   ClusterIP   10.99.136.212    <none>        8083/TCP                     2m12s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-dex-server    1/1     1            1           2m12s
deployment.apps/argocd-redis         1/1     1            1           2m12s
deployment.apps/argocd-repo-server   1/1     1            1           2m11s
deployment.apps/argocd-server        1/1     1            1           2m11s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-dex-server-66bc48cb59   1         1         1       2m12s
replicaset.apps/argocd-redis-6fb68d9df5        1         1         1       2m12s
replicaset.apps/argocd-repo-server-5bc4ff7b6   1         1         1       2m11s
replicaset.apps/argocd-server-68f79c4cf9       1         1         1       2m11s

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     2m11s
```

모든 오브젝트가 올라올때까지 기다려야한다. 

### ArgoCD 서버에 접근하기. 

이제는 ArgoCD 에 접근하기 위해서 변경해야할 사항이 있다. 

```shell
service/argocd-server           ClusterIP   10.96.1.86       <none>        80/TCP,443/TCP               2m12s
```

보는바와 같이 argocd-server 은 ClusterIP 설정되어 외부에서 접근이 안된다. 이를 NodePort 로 변경해 주어야한다. 

```shell
kubectl -n argocd edit svc argocd-server

... 생략 
spec:
  clusterIP: 10.96.1.86
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 8080
  - name: https
    port: 443
    protocol: TCP
    targetPort: 8080
  selector:
    app.kubernetes.io/name: argocd-server
  sessionAffinity: None
  type: NodePort <-- ClusterIP 에서 NodePort 로 바꿔주었다. 
status:
  loadBalancer: {}

```

저장하고 나가면 다음과 같다. 

```shell
service/argocd-server           NodePort    10.96.1.86       <none>        80:31602/TCP,443:31990/TCP   6m11s
```

이제 NodePort 로 바뀌었따. 이제 서버에 접근해보자. 

### 노드 주소 찾기. 

```shell
kubectl get nodes -o wide

NAME             STATUS   ROLES    AGE    VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION      CONTAINER-RUNTIME
docker-desktop   Ready    master   3d8h   v1.19.3   192.168.65.3   <none>        Docker Desktop   4.19.121-linuxkit   docker://20.10.2
```

우리는 docker-desktop 한대의 서버에 설치 했으니 1개의 노드 정보가 나온다. 

localhost(docker-desktop이므로 가능) 으로 접근이 가능하며 argoCd의 포트는 http는 31602, https는 31990 임을 이미 봤다. 

https://localhost:31602/ 으로 접근하면 다음과 같다. 

![argocd03](https://user-images.githubusercontent.com/66154381/109144827-09a43e00-77a5-11eb-8387-239d507cfb6d.png)

localhost(안전하지 않음) 으로 이동을 클릭한다. 

그럼 다음 화면가 같이 나온다. 초기 정보는 다음과 같다. 

![argocd04](https://user-images.githubusercontent.com/66154381/109145130-6acc1180-77a5-11eb-8fd7-504f929fc009.png)

- Username: admin
- Password: pod이름 (argocd-server-68f79c4cf9-pkcv6) 을 넣어보자. 

SignIn 을 클릭하면 초기 화면으로 들어간다. 

![argocd05](https://user-images.githubusercontent.com/66154381/109145220-8df6c100-77a5-11eb-8272-5e811efa1aa2.png)

## 결론

지금까지 ArgoCD 를 설치해 보았다. 

docker-desktop 으로 작업해 보았지만 설치 과정은 그리 어렵지 않음을 알 수 있다. 

이제부터 ArgoCD로 고고~..