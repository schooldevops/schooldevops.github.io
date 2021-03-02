---
layout: post
title:  "Kubernetes ArgoCD 간단한 웹 프로그래밍 개발 및 DockerHub 업로드"
date:   2021-03-02 14:58:49 +0900
categories: Kubernetes
tags: [kubernetes, argocd, ci, cd, gitops, deploy, docker, dockerhub]
toc: true
---

# 간단한 웹 프로그래밍 개발 및 DockerHub 업로드

이제는 간단한 Hello World 웹 어플리케이션을 만들어 보자. 

우리 예제는 [go gin](https://github.com/gin-gonic/gin#installation) 을 이용하여 개발해 볼 것이다. 


## go module 설치 

```go
go mod init com.github.schooldevops.go.gin

go: creating new go.mod: module com.github.schooldevops.go.gin
```

## Gin 의존성 가져오기 

```go
go get -u github.com/gin-gonic/gin

...
```

위와 같이 필요한 모듈을 가져왔다. 

## 간단한 샘플 작성하기. 

main.go 파일을 열어 다음 내용을 추가한다. 

```go
package main

import (
	"bufio"
	"net/http"
	"os"
	"strings"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()

	//	헬스 체크 수행하기. 
	r.GET("/health", helathCheck)

	//	:greet 에 해당하는 path parameter 를 획득하여 인사를 반환한다. 
	r.GET("/hello/:greet", greeting)

	r.Run()
}

func helathCheck(c *gin.Context) {

	line := readFile("health")
	if strings.Contains(string(line), "OK") {
		c.String(http.StatusOK, string(line))
	} else {
		c.String(http.StatusInternalServerError, string(line))
	}
}

func readFile(fileName string) string {

	fo, err := os.Open("health")
	if err != nil {
		return ""
	}

	reader := bufio.NewReader(fo)
	for {
		line, isPrefix, err := reader.ReadLine()
		if isPrefix || err != nil {
			break;
		} else {
			return string(line)
		}
	}

	return ""
}

func greeting(c *gin.Context) {
	greet := c.Param("greet")
	c.String(http.StatusOK, "Hello " + greet)
}
```

위 코드는 2개의 REST API 를 제공한다. 

- /helath : 헬스체크 요청에 대한 응답을 수행한다. 
- /hello/:greet : :greet 값을 받아서 Hello <인사> 말을 응답한다. 

### 테스트 수행하기. 

기본적으로 gin 은 8080으로 서비스가 된다. 

```go
curl http://localhost:8080/helath
OK

curl http://localhost:8080/hello/kido
Hello kido
```

## Docker 파일 작성하기. 

Dockerfile 을 생성하고 다음과 같이 작성한다. 

```go
FROM golang:1.12 AS build

WORKDIR /src/
COPY main.go /src/
COPY go.mod go.sum /src/
RUN go mod download
RUN CGO_ENABLED=0 go build -o /bin/webserver

FROM alpine:latest
COPY --from=build /bin/webserver /bin/webserver
COPY health /bin/
WORKDIR /bin/
ENTRYPOINT ["/bin/webserver"]
```

### 빌드 및 Docker hub push 하기. 

#### 빌드하기 

필자의 docker hub 의 계정은 unclebae 이므로 이미지 이름을 작성할 때 unclebae/ 로 작성했다. 

```go
docker build -t unclebae/gogreet:v1.0 .
```

#### Docker login 수행하기. 

```go
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: unclebae
Password: 
Login Succeeded
```

#### 푸시하기

```go
docker push unclebae/gogreet:v1.0

The push refers to repository [docker.io/unclebae/gogreet]
5f70bf18a086: Pushed 
d79e9e023899: Pushed 
625f475afe8e: Pushed 
cb381a32b229: Mounted from library/alpine 
v1.0: digest: sha256:a96e03a55b71ea55db3186fbf5ac612593d84d582e853fce2d312801fa2bf5f3 size: 1152
```

![go-web-docker](https://user-images.githubusercontent.com/66154381/109616402-725e3280-7b78-11eb-9ecf-2872b360ed26.png)

위와 같이 docker hub 에 정상으로 push 되었음을 확인할 수 있다. 

