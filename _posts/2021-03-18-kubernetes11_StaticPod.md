---
layout: post
title:  "Kubernetes StaticPod 실습  "
date:   2021-03-16 11:45:49 +0900
categories: Kubernetes
tags: [Kubernetes, Kubectl, StaticPod]
toc: true
---

# StaticPod 실습

- StaticPod 는 Kubelet 에 의해서 실행되는 POD 이다. 
- kube-contolplane 의 특정 디렉토리에 설정해 두면, kubelet 이 자동으로 실행해준다. 

## staticPOD 확인하기. 

```go
kubectl get pods --all-namespaces | grep controlplane

kube-system   etcd-controlplane                      1/1     Running   0          57m
kube-system   kube-apiserver-controlplane            1/1     Running   0          57m
kube-system   kube-controller-manager-controlplane   1/1     Running   0          57m
kube-system   kube-scheduler-controlplane            1/1     Running   0          57m

```

controlplane 가 붙어 있는 pods 는 static pod 로 동작한다. 

마스터 노드의 이름에 해당하는 이름이 controlplane 이다. 

## 실행되는 node 확인하기. 

```go
get pods --all-namespaces -o wide | grep controlplane

kube-system   coredns-f9fd979d6-kg5w6                1/1     Running   0          60m   10.244.0.3    controlplane   <none>           <none>
kube-system   coredns-f9fd979d6-pkzll                1/1     Running   0          60m   10.244.0.2    controlplane   <none>           <none>
kube-system   etcd-controlplane                      1/1     Running   0          60m   172.17.0.8    controlplane   <none>           <none>
kube-system   kube-apiserver-controlplane            1/1     Running   0          60m   172.17.0.8    controlplane   <none>           <none>
kube-system   kube-controller-manager-controlplane   1/1     Running   0          60m   172.17.0.8    controlplane   <none>           <none>
kube-system   kube-flannel-ds-amd64-s9z7b            1/1     Running   0          60m   172.17.0.8    controlplane   <none>           <none>
kube-system   kube-proxy-4pxhm                       1/1     Running   0          60m   172.17.0.8    controlplane   <none>           <none>
kube-system   kube-scheduler-controlplane            1/1     Running   0          60m   172.17.0.8    controlplane   <none>           <none>
```

모두 controlplane 에 떠 있음을 확인할 수 있다. 

## StaticPod 메니페스트 파일 위치 찾기. 

```go
ps -aux | grep kubelet 

... 커맨드에 나오는 내용중 --config= 부분을 찾는다. 
```

### config 내용 살펴보기

/var/lib/kubelet/config.yaml 파일 내용을 살펴본다, 

```go
cat /var/lib/kubelet/config.yaml

apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
resolvConf: /run/systemd/resolve/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 0s

...
staticPodPath: /etc/kubernetes/manifests
...

streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
```

위 내용중 staticPodPath 디렉토리를 확인하면 된다. 

```go
ls -alt /etc/kubernetes/manifests

total 24
drwx------ 2 root root 4096 Mar 16 03:47 .
-rw------- 1 root root 2090 Mar 16 03:47 etcd.yaml
-rw------- 1 root root 3345 Mar 16 03:47 kube-controller-manager.yaml
-rw------- 1 root root 1384 Mar 16 03:47 kube-scheduler.yaml
-rw------- 1 root root 3658 Mar 16 03:47 kube-apiserver.yaml
drwxr-xr-x 4 root root 4096 Mar 16 03:47 ..
```

staticPod 의 정보를 확인할 수 있다. 

- etcd
- kube-controller-manager
- kube-scheduler
- kube-apiserver

위 4개의 pod 는 기본적으로 staticPod 임을 확인할 수 있다. 

### Static Pod manifest 파일 살펴보기. 

kube-apiserver.yaml 파일을 살펴보자. 

```go
cat /etc/kubernetes/manifests/kube-apiserver.yaml 

apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 172.17.0.8:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=172.17.0.8
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.19.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 172.17.0.8
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 172.17.0.8
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 172.17.0.8
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}
```

## Customer Static Pod 생성하기. 

### Pod Manifest 생성하기. 

```go
kubectl run static-busybox --image=busybox --dry-run=client -o yaml > busybox-pod.yaml
```

### Pod 내용 적절히 수정하고 staticPod manifest 디렉토리오 이관하기. 

```go
cp busybox-pod.yaml /etc/kubernetes/manifests/busybox-pod.yaml
```

### pod 내용 살펴보기. 

```go
kubectl get pods --all-namespaces

NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
default       static-busybox-controlplane            1/1     Running   0          33s
kube-system   coredns-f9fd979d6-kg5w6                1/1     Running   0          75m
kube-system   coredns-f9fd979d6-pkzll                1/1     Running   0          75m
kube-system   etcd-controlplane                      1/1     Running   0          75m
kube-system   kube-apiserver-controlplane            1/1     Running   0          75m
...
```

static-busybox-controlplane 이 생성되었음을 확인할 수 있다. 

## staticPod 삭제

### 우선 staticPod 가 존재하는 node 확인하기. 

```go
kubectl get pods -o wide 
```

### 노드를 확인하면 ssh 로 접근하기.

ssh <node_ip> 

### kubelet 의 파라미터 정보 확인하기. 

```go
ps -aux | grep kubelet

... config=
```

config= 의 경로 찾아서 내용 보기. 

### staticPodPath 경로를 찾고, manifest 파일을 제거한다. 

config 파일 내용에서 staticPodPath 경롤르 찾아서 삭제할 staticPod manifest 제거한다. 

### pod 삭제하기

수행중인 pod 를 삭제한다. 

```go
kubectl delete pods <podname>
```
