---
layout: post
title:  "Docker 컨테이너 가볍게 작성하기"
date:   2021-02-24 14:00:49 +0900
categories: Docker
tags: [Go, Docker, Container, Build]
toc: true
---

# Docker 컨테이너 가볍게 빌드하기.

Docker는 컨테이너 툴은 현재 가장 많이 사용되고 있는 도구이다. 

컨테이너 이미지를 작성할때, 어떻게 작성하느냐에 따라서 컨테이너 빌드 속도와, 빌드 후 이미지 크기에 영향을 주며, 이러한 영향은 프로젝트 개발 라이프 사이클에서 어느정도 영향을 주게 된다. 

필자의 경우 Docker Build 시 Dockerfile 을 비효율적으로 작성하는 바람에 빌드 시간이 30분이 소요된 경우도 있었다. 

이런 이유는 Docker Build 레이어를 사용한다는 사실을 모른채 Dockerfile을 작성했었고, 매번 빌드시마다 의존성 파일을 다운로드하는 비효율적인 빌드를 수행했었기 때문에 발생했었다. 

이번 아티클은 Golang 으로 웹 어플리케이션을 간단히 작성해보고, Docker 이미지 빌드를 수행하는 방법에 대해서 알아볼 것이다. 

## Go web 프로그램 작성하기. 

간단한 Go Web 프로그램을 작성해보자. 

### Go Mod 실행하기. 

Go에서는 Go mod 라는 커맨드를 이용하여 모듈 의존성을 관리하도록 하고 있다. 일종의 node 모듈과 유사한 개념이라고 보면 된다. 

```
go mod init com.github.schooldevops.go.sample
```

이렇게 수행하면 go.mod 파일과 go.sum 파일이 생성이 된다. 여기서 go.mod 파일이 의존성을 관리하는 파일이다. 

### 환경 변수를 읽어 오기 위한 의존성 가져오기. 

우리는 간단한 웹 프로그램에서 환경 변수를 활용하기 위해서 go 의존성 라이브러리를 임포트 할 것이다. 

그러므로 다음과 같이 의존성 모듈을 가져오자. 

```
go get github.com/joho/godotenv
```

이렇게 의존성을 가져오면 go.mod 파일이 다음과 같이 의존성 라이브러리가 추가된 것을 확인할 수 있다. 

go.mod

```
module com.github.schooldevops.go.sample

go 1.15

require github.com/joho/godotenv v1.3.0 // indirect

```

### 시스템 점검 화면을 나타내기 위한 html 코드 작성하기. 

index.html 파일을 생성하고 다음과 같이 작성하자. 

```html

<!doctype html>
<html lang="en" class="h-100">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <meta name="author" content="Mark Otto, Jacob Thornton, and Bootstrap contributors">
    <meta name="generator" content="Hugo 0.79.0">
    <title>{{.Title}}</title>

    <link rel="canonical" href="https://getbootstrap.com/docs/5.0/examples/cover/">

    

    <!-- Bootstrap core CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-giJF6kkoqNQ00vy+HMDP7azOuL0xtbfIcaT9wjKHr8RbDVddVHyTfAAsrekwKmP1" crossorigin="anonymous">

    <!-- Favicons -->
<meta name="theme-color" content="#7952b3">


    <style>
      .bd-placeholder-img {
        font-size: 1.125rem;
        text-anchor: middle;
        -webkit-user-select: none;
        -moz-user-select: none;
        user-select: none;
      }

      @media (min-width: 768px) {
        .bd-placeholder-img-lg {
          font-size: 3.5rem;
        }
      }
    </style>

    
    <!-- Custom styles for this template -->
    <link href="cover.css" rel="stylesheet">
  </head>
  <body class="d-flex h-100 text-center text-white bg-dark">
    
<div class="cover-container d-flex w-100 h-100 p-3 mx-auto flex-column">
  <header class="mb-auto">
    <div>
      <h3 class="float-md-start mb-0">{{.SystemName}}</h3>
    </div>
  </header>

  <main class="px-3">
    <h1>{{.Title}}</h1>
    <br/>
    <pre class="lead">{{.Contents}}</pre>

    {{if .Period}}
      <div class="lead">{{.StartTime}} ~ {{.EndTime}}</div>
    {{end}}
  </main>

  <footer class="mt-auto text-white-50">
    <p>Cover template for <a href="https://getbootstrap.com/" class="text-white">Bootstrap</a>, by <a href="https://twitter.com/mdo" class="text-white">@mdo</a>.</p>
  </footer>
</div>

    <!-- Option 1: Bootstrap Bundle with Popper -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta1/dist/js/bootstrap.bundle.min.js" integrity="sha384-ygbV9kiqUc6oa4msXn9868pTtWMgiQaeYH7/t7LECLbyPA2x65Kgf80OJFdroafW" crossorigin="anonymous"></script>

    <!-- Option 2: Separate Popper and Bootstrap JS -->
    <!--
    <script src="https://cdn.jsdelivr.net/npm/@popperjs/core@2.5.4/dist/umd/popper.min.js" integrity="sha384-q2kxQ16AaE6UbzuKqyBE9/u/KzioAlnx2maXQHiDX9d4/zp8Ok3f+M7DPm+Ib6IU" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta1/dist/js/bootstrap.min.js" integrity="sha384-pQQkAEnwaBkjpqZ8RU1fF1AKtTcHJwFl3pblpTlHXybJjHpMYo79HY3hIi4NKxyj" crossorigin="anonymous"></script>
    -->
    
  </body>
</html>

```

bootstrap 을 이용할 것이기 때문에 bootstrap 을 위한 css 와 javascript 을 가져오기 위한 코드가 들어 있다. 

그리고 go 에서 제공하는 내장 라이브러리인 template 을 이용하여 필요한 값을 채워넣기 위해서 다음과 같은 템플릿 플레이스홀더 코드를 작성했다. 

- {{.Title}}
- {{.SystemName}}
- {{.Contents}}
- {{if .Period}} ~ {{end}}
- {{.StartTime}} ~ {{.EndTime}}

이 속성들은 다음 경로에서 문서를 참조하자. [https://golang.org/pkg/text/template/](https://golang.org/pkg/text/template/)

용도는 프로그램 내에서 해당 설정 파일을 읽어서, 시스템 점검을 위한 변수 값들을 위 템플릿 플레이스홀더 부분에 교체하는 것이다. 

### 설정파일 작성하기

이제는 설정파일을 작성해보자. 

설정파일은 위 플레이스홀더 코드에 대체될 정보들을 담고 있다. 

envfile 이라는 파일을 만들고 다음과 같이 작성하자. 

```conf
# .env file

SITE_NAME=SCHOOLDEVOPS
TITLE=시스템 점검중
CONTENTS=시스템 점검중에 있습니다.
PERIOD=true
START_TIME=2021-01-24 00:00:00
END_TIME=2021-01-28 00:00:00
```

보는바와 같이 위 플레이스홀더 에 매핑되도록 설정값이 들어 있다. 

### 웹 서버 코드 작성하기. 

이제 요청을 받아 들이는 서버 코드를 작성하자. 

- http://localhost:8888 로 요청이 들어오면 index.html 에 플레이스홀더가 채워진 상태로 시스템 점검 페이지를 보여주게 된다.

main.go 파일을 아래와 같이 작성해주자. 

```go
package main

import (
	"fmt"
	"html/template"
	"log"
	"net/http"
	"os"
	"strconv"

	"github.com/joho/godotenv"
)

type Noti struct {
	SystemName string //	시스템 이름
	Title      string //	제목
	Contents   string //	내용
	Period     bool   // 	기간 유무
	StartTime  string //	시작시간
	EndTime    string //	종료시간
}

func Index(w http.ResponseWriter, r *http.Request) {
	tmpl := template.Must(template.ParseFiles("index.html"))

	err := godotenv.Load("envfile")
	if err != nil {
		log.Fatalf("Error loading envfile. %s", err)
	}

	isShowPeriod, _ := strconv.ParseBool(os.Getenv("PERIOD"))

	data := Noti{
		SystemName: os.Getenv("SITE_NAME"),
		Title:      os.Getenv("TITLE"),
		Contents:   os.Getenv("CONTENTS"),
		Period:     isShowPeriod,
		StartTime:  os.Getenv("START_TIME"),
		EndTime:    os.Getenv("END_TIME"),
	}

	tmpl.Execute(w, data)
}

func main() {
	http.HandleFunc("/", Index)
	fmt.Print("Start Server port on 8888")
	log.Fatal(http.ListenAndServe(":8888", nil))
}

```

코드는 매우 간단하다. 

main 함수를 살펴보면 다음과 같다. 

```go
//  핸들러 함수를 등록한다. 요청이 '/' 로 들어오면 Index 라는 핸들러 함수를 호출하는 구조이다. 
	http.HandleFunc("/", Index)
//  서버가 실행될때 콘솔로 노출된다. 
	fmt.Print("Start Server port on 8888")
//  서버를 띄우고, 8888포트로 리슨하게 된다.     
	log.Fatal(http.ListenAndServe(":8888", nil))
```

그리고 핸들러 코드는 다음과 같이 작성된다. 

```go
func Index(w http.ResponseWriter, r *http.Request) {
	tmpl := template.Must(template.ParseFiles("index.html"))

	err := godotenv.Load("envfile")
	if err != nil {
		log.Fatalf("Error loading envfile. %s", err)
	}

	isShowPeriod, _ := strconv.ParseBool(os.Getenv("PERIOD"))

	data := Noti{
		SystemName: os.Getenv("SITE_NAME"),
		Title:      os.Getenv("TITLE"),
		Contents:   os.Getenv("CONTENTS"),
		Period:     isShowPeriod,
		StartTime:  os.Getenv("START_TIME"),
		EndTime:    os.Getenv("END_TIME"),
	}

	tmpl.Execute(w, data)
}
```

위 코드는 단순히 index.html 파일을 읽고, envfile 을 읽어서 index.html 파일에 존재하는 플에이스 홀더에 반영하는 template 작업을 수행하는 코드이다. 

### 실행하기. 

```log
go run main.go

Start Server port on 8888
```

가 출력이 되며 화면에서 요청을 하면 다음과 같은 화면이 노출이 된다. 

위 코드를 실제 브라우저에서 살펴보면 다음과 같다. 

![systemcheck](https://user-images.githubusercontent.com/66154381/108952376-b353d400-76ac-11eb-821c-ebd68fd3967a.png)

## Dockerfile 작성하기

이제 실제 컨테이너 이미지를 생성해보자. 

Dockerfile 을 다음과 같이 작성해준다. 

```go
FROM golang:1.12 AS build

WORKDIR /src/
COPY main.go /src/
COPY go.mod go.sum /src/
COPY index.html /src/
COPY envfile /src/

RUN go mod download
RUN CGO_ENABLED=0 go build -o demo

ENTRYPOINT ["demo"]

```

소스 코드, mod 파일, index.html 파일 envfile 등 대부분 파일을 Docker 이미지에 추가했다. 

그리고 한가지 더 중요한 것은 golang:1.2 에서 사용한 base 이미지 와 의존성 파일 모두 이미지에 추가된다는 것이다. 

### Docker build 하기. 

```bash
docker build -t system-check:v1.1 .
```

위 내용을 실행하면 필요한 docker 베이스 이미지와, 레이어 이미지들을 다운로드 받게 된다. 


```bash
docker images | grep system-check
system-check                         v1.0                                                    b78bb8f0fc24   31 seconds ago   829MB
```

생성된 이미지 크기는 829MB 이다. 작은 프로그램 하나가 크기가 엄청나다. 

### 실행하기

```
docker run -d -p 9999:8888 system-check:v1.0 
```

실행결과 정상적으로 위 화면과 같은 결과를 보여준다. 

### 이미지 크기 줄이기

이제는 이미지 크기를 줄여보자. 829MB 를 컴팩트하게 줄이는 작업을 할 것이다. 

go 의 경우 실제 빌드하면 실행파일 한개만 떨어진다. 실행파일 1개 분량으로 크기를 줄일 수 있다. 

Dockerfile 다음과 같이 수정하자. 

```go
FROM golang:1.12 AS build

WORKDIR /src/
COPY main.go /src/
COPY go.mod go.sum /src/
RUN go mod download
RUN CGO_ENABLED=0 go build -o /bin/demo

FROM scratch
COPY --from=build /bin/demo /bin/demo
COPY index.html /bin/
COPY envfile /bin/
WORKDIR /bin/
ENTRYPOINT ["/bin/demo"]
```

무언가 달라진것이 보이는가? 

이전에는 베이스 이미지를 가져와서 Docker 이미지 내에 /src 디렉토리에 모두 밀어넣었다면, 이번 작업은 2단계로 나눠서 처리한다. 

- 1단계:
    - base이미지인 golang:1.12 를 가져혼다. 그리고 이 단계 이름을 build 로 잡았다. 
    - 작업 디렉토리를 생성하고, 컴파일에 필요한 파일들을 이관한다. 
    - 그리고 빌드를 수행하여 최종 실행파일을 만든다. 
- 2단계:
    - 빈 이미지를 만든다. 이는 베이스 이미지가 없이 매우 가볍게 가져간다. 
    - build 스테이지에서 만든 이미지를 /bin/demo 로 복사한다. 
    - 필요한 파일 (실행에 꼭 필요한파일) 을 복사한다. index.html, envfile 이 /bin/ 디렉토리로 복사된다. 
    - 실행 위치로 이동한다. 
    - ENTRYPOINT 로 생성된 실행파일을 실행하게 한다. 

위 과정과 같이 1단계 빌드 단계의 내용은 실제 이미지로 생성이 되지 않고, 2단계의 내용만 이미지로 들어가므로 컴팩트한 결과가 나오게 된다. 

### 이미지 빌드하기. 

```bash
docker build -t system-check:v1.1 .
```

이미지 태그만 v1.0 에서 1.2f로 달아주었다.

### 생성된 결과 비교하기. 

```bash
system-check                         v1.0                                                    b78bb8f0fc24   9 minutes ago    829MB
system-check                         v1.1                                                    e78fc2c41621   19 minutes ago   10.1MB
```

어떤가 이번에는 10.1Mb 로 829MB에 비하면 엄청난 크기 차이가 난다. 

## 결론

지금까지 간단한 웹 프로그램을 작성했고, Docker 이미지도 생성해 보았다. 

어떻게 Docker 이미지를 생성 하느냐에 따라서 이미지의 크기가 매우 큰 차이가 남을 확인했다. 

실제 현업에서 이미지의 차이는 빌드, 배포 시간의 단축을 가져오고, 이미지 리포지토리의 크기도 매우 줄여주게 될 것이다. 

Cloud 를 사용하는 환경이라면, 위 결과를 통해 비용역시 절감할 수 있을 것이다. 

아무쪼록 컨테이너 이미지는 작을수록 좋다는 것이므로 꼭 시도해보자. 

관련 코드는 다음에서 확인할 수 있다. ~[https://github.com/schooldevops/go_web_sitecheck](https://github.com/schooldevops/go_web_sitecheck)



