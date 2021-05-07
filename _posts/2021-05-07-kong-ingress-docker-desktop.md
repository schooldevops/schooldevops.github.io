---
layout: post
title:  "Tutorial: Using Kong Kubernetes Ingress Controller as an API Gateway"
date:   2021-05-07 08:45:49 +0900
categories: K8s
tags: [APIGW, API Gateway, Kong, K8S, Ingress]
toc: true
---

# Kong Ingress Controller for Kubernetes 사용하기

여기서는 [Kong공식 가이드](https://konghq.com/blog/kubernetes-ingress-api-gateway) 를 바탕으로 Local 의 Docker Desktop 을 활용하여 Kong Ingress Controller 을 설치하여 API Gateway 서비스를 수행해 볼 것이다. 

## 간단한 REST API POD 생성하기. . 

### 간단한 REST API Service 작성

foo.py

```python
from flask import Flask
app = Flask(__name__)
 
@app.route('/foo')
def hello():
    return '{"msg":"Hello from the foo microservice"}'
 
if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0')
```

간단하게 Flask 를 활용하여 /foo 엔드포인트에 대해서 JSON 포맷의 메시지를 내려 보내는 API 이다. 

### Dockerfile 작성하기. 

```Dockerfile
FROM python:3-alpine
 
WORKDIR /app
 
RUN echo "Flask==1.1.1" > requirements.txt
RUN pip install -r requirements.txt
COPY foo.py .
 
EXPOSE 5000
 
CMD ["python", "foo.py"]
```

위와 같이 bar 역시 동일하게 작업한다. 

관련 샘플은 이미 Docker hub에 올라와 있기 때문에 우린 그것을 그대로 이용할 것이다. 

## Service 배포 하기. 

이제는 Docker desktop K8S 에 Service 를 배포해 보자. 

foo.yaml 을 다음과 같이 작성한다. 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: foo
  template:
    metadata:
      labels:
        app: foo
    spec:
      containers:
      - name: api
        image: digitalronin/foo-microservice:0.1
        ports:
        - containerPort: 5000
---
apiVersion: v1
kind: Service
metadata:
  name: foo-service
  labels:
    app: foo-service
spec:
  ports:
  - port: 5000
    name: http
    targetPort: 5000
  selector:
    app: foo
```

보는 바와 같이 Deployment 를 우선 작성했다. 

- replicas: 1 오직 하나의 pod 만 실행하도록 했다. 
- image: digital....  이 부분은 이미 존재하는 Docker Hub 에서 이미지를 가져올 것이다. 
- port: 포트는 5000으로 K8S에 노출할 포트를 서비스에 정의했다. 
- targetPort: 컨테이너 포트 역시 5000 설정했다. 
- 즉, 서비스 타입을 지정하지 않았으므로 ClusterIP 를 통한 연결을 수행한다. 

### 배포

```
kubectl apply -f foo.yaml
kubectl apply -f bar.yaml
```

foo/bar 2개의 Deployment 를 배포 했다. 

### 테스트하기. 

테스트를 위해서는 port-forward 를 수행해서 테스트 해 볼 것이다. 

```sh
kubectl port-forward service/foo-service 5000:5000
```

위와 같이 port-forwarding 을 수행했다. 

```
curl http://localhost:5000/foo

{"msg":"Hello from the foo microservice"}
```

정상으로 결과가 넘어오는 것을 알 수 있다. 

bar 역시 동일한 방법으로 배포 해 보자. 


## Install Kong for Kubernetes

이제는 K8S Ingress Controller 를 설치할 것이다. 

Ingress 는 외부에서 K8S 로 요청을 전달하는 API Gateway 역할을 수행한다. 이를 위해서는 Ingress Controller 가 필요하며, 대표적으로 Ngins Ingress Controller 가 있다. 

우리는 Kong 을 Ingress Controller 로 이용할 것이므로 아래와 같이 Kong Ingress Controller 를 설치할 것이다. 

```
kubectl create -f https://bit.ly/k4k8s
```

### 결과보기 

```
...
service/kong-proxy created
service/kong-validation-webhook created
deployment.apps/ingress-kong created
```

### Check Kong Ingress Controller 정보 살펴보기 

```
kubectl -n kong get service kong-proxy

NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
kong-proxy   LoadBalancer   10.107.46.227   localhost     80:30238/TCP,443:31353/TCP   26s
```

### Another Objects for Kong Ingress Controller

이제는 kong 네임스페이스에 설치된 각 객체들을 확인해보자. 

```
kubectl -n kong get all

NAME                                READY   STATUS    RESTARTS   AGE
pod/ingress-kong-58b996687d-jxcbl   2/2     Running   0          56s

NAME                              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/kong-proxy                LoadBalancer   10.107.46.227   localhost     80:30238/TCP,443:31353/TCP   56s
service/kong-validation-webhook   ClusterIP      10.111.61.154   <none>        443/TCP                      56s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-kong   1/1     1            1           56s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-kong-58b996687d   1         1         1       56s
```

보는 바와 같이 ingress-kong 컨트롤러를 위한 pod가 떠있다. 

서비스는 kong-proxy, kong-validation-webhook 가 떠있다. kong-proxy 는 LoadBalancer임을 알 수 있으며, 우리는 EXTERNAL-IP 를 통해서 접근할 수 있다. localhost 라는 것을 확인할 수 있다. 

제공되는 port 는 80, 443 포트로 접속이 가능하다. 

deployment 는 ingress-kong 이 동작하며, replicaset 역시 ingress-kong-xxx 으로 1개의 pod를 관리함을 알 수 있다. 

### Kong Ingress Controller 테스트 하기.

```
curl http://localhost

{"message":"no Route matched with those values"}
```

위와 같이 아직 라우트 설정을 하지 않았음을 알 수 있다. 즉 요청에 대해서 처리할 대상을 설정해 주지 않았기 때문이다. 

## Kong Config 수행하기. 

이제 요청에 대해서, K8S 서비스로 요청을 전달할 수 있도록 Kong 설정을 해보자. 

foo-ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: foo
  namespace: default
 
spec:
  ingressClassName: kong
  rules:
  - http:
      paths:
      - path: /foo
        pathType: Prefix
        backend:
          service:
            name: foo-service
            port:
              number: 5000
```

apiVersion은 networking.k8s.io/v1 에 있는 버젼을 사용한다. 

kind: Ingress 이므로 우리는 들어오는 요청에 대한 라우팅 정의를 할 것이다. 

rules.http.paths.path: /foo 요청이 /foo로 들어온다면 foo-service 이전에 지정한 서비스로 전달할 것이다. 

해당 서비스는 포트 번호가 5000 이다. 

### ingress 배포하기. 

```
kubectl apply -f foo-ingress.yaml
kubectl apply -f bar-ingress.yaml
```

### 테스트하기. 

```
curl http://localhost/foo

{"msg":"Hello from the foo microservice"}

curl http://localhost/bar

{"msg":"Hello from the bar microservice"}
```

## Wrap Up

지금까지 Local Docker Desktop 환경에서 Kong Ingress Controller 를 활용하여, 요청을 적절한 서비스로 라우팅 했다. 

- 우선 간단한 서비스를 만든다.
- Dockerfile 을 통해서 Docker 이미지를 생성하고, Docker Hub에 등록한다. 
- 등록된 Docker 이미지를 활용하여 Deployment 메니페스트 파일로 K8S에 배포한다. 
- Kong Ingress Controller 를 설치했다. 네임스페이스는 kong 였음을 확인했다. 
- Kong Ingress Config 를 지정하고, 배포를 수행했다. 
- /foo요청은 foo-service 로 요청을 전달하고, /bar요청은 bar-service로 요청을 전달하도록 설정했다. 
- 테스트를 통해서 정상적인 결과를 확인할 수 있게 되었다. 

이렇게 Kong Ingress 로 MSA 의 필수 요소인 API Gateway와 K8S 연동을 해 보았다. 의외로 간단하다는 것을 알 수 있었다. 
