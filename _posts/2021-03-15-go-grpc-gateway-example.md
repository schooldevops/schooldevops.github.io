---
layout: post
title:  "Go Grpc with REST API"
date:   2021-03-15 08:45:49 +0900
categories: gRpc
tags: [grpc, go, rest api, gateway]
toc: true
---

# grpc with API Gateway

Grpc 와 API Gateway 를 이용하여 REST API 지원하기. 

## initial go module

```go
go mod init echo-grpc
```

## install module

```go
go get "github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway"
go get "github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2"
go get "google.golang.org/grpc/cmd/protoc-gen-go-grpc"
go get "google.golang.org/protobuf/cmd/protoc-gen-go"
```

## Copy google api for grpc gateway

https://github.com/googleapis/googleapis 에서 git 별도의 디렉토리에 clone 한다. 

파일중 google/api 디렉토리의 내용을 복사하여 자신의 프로젝트 워크스페이스에 옮겨 놓는다. 

필요한 파일은 다음과 같다. 

```go
google/api/annotations.proto
http.proto
```

위 파일들은 모두 grpc에 rest api 를 지원하기 위한 필수 파일이다. 


## proto 파일 작성. 

워크스페이스에서 proto/echopb 디렉토리를 생성하고 echo_server.proto 파일을 생성한다. 

```go
syntax = "proto3";

package proto;
option go_package = "echopb";

import "google/api/annotations.proto";

message EchoRequest {
	string message = 1;
}

message EchoResponse {
	string result = 1;
}

service EchoService {
	rpc Echo(EchoRequest) returns (EchoResponse) {
		option (google.api.http) = {
			get: "/v1/echo/{message}"
		};
	}
}
```

위와 같이 매우 단순한 proto 파일이며, grpc Echo 메소드에 rest api 를 적용한 것이다. 

## proto pb 파일 생성하기. 

### grpc server/client 인터페이스 스텁을 생성하기. 

```go
protoc -I . \
  --go_out . \
  --go_opt paths=source_relative \
  --go-grpc_out . --go-grpc_opt paths=source_relative \
  proto/echopb/echo_server.proto
```

### rest api 용 인터페이스 스텁 생성하기. 

```go
protoc -I . \
    --grpc-gateway_out . \
    --grpc-gateway_opt logtostderr=true \
    --grpc-gateway_opt paths=source_relative \
  proto/echopb/echo_server.proto

```

위 커맨드를 각각 수행하면 proto/echopb 디렉토리 하위에 pb파일이 생성이 될 것이다. 

- echo_server.pb.go: 에코 서버를 위한 기본적인 모델이 정의된다. 
- echo_server_grpc.pb.go: grpc 서버를 위한 인터페이스를 정의한다.
- echo_server_pb.gw.go: rest api 서버를 위한 인터페이슬 정의한다. 

## grpc 전용 서버 개발하기. 

이제는 grpc 서버를 직접 작성하자. 이때 proto/echopb 내에 존재하는 echo_server_grpc.pb.go 파일의 인터페이스를 이용하여 서버를 구현할 것이다. 

```go
package main

import (
	"context"
	"echo-grpc/proto/echopb"
	"fmt"
	"log"
	"net"

	"google.golang.org/grpc"
)

const portNumber = "9001"

type server struct{
	echopb.EchoServiceServer
}

func main() {
	lis, err := net.Listen("tcp", ":"+portNumber)
	if err != nil {
		log.Fatalf("Fail to listen: %v\n", err)
	}

	opts := []grpc.ServerOption{}
	s := grpc.NewServer(opts...)

	echopb.RegisterEchoServiceServer(s, &server{})

	log.Printf("start grpc server on %s port", portNumber)
	if err := s.Serve(lis); err != nil {
		log.Fatalf("Failed to server: %v", err)
	}
}

func (*server) Echo(ctx context.Context, req *echopb.EchoRequest) (*echopb.EchoResponse, error) {
	message := req.GetMessage()
	fmt.Printf("Echo %v\n", message)

	return &echopb.EchoResponse{
		Result: "Hello " + message,
	}, nil
	
}

```

- grpc 서버는 9001 포트로 기동이 된다. 
- echopb.RegisterEchoServiceServer(s, &server{}): grpc 서비스를 사용하겠다고 등록한다. 
- func (*server) Echo(ctx context.Context, req *echopb.EchoReques... : 이 부분은 실제 grpc 서비스를 구현한다. 

위와 같이 작업하먄 grpc echo 서버는 구현이 되었다. 

### 실행하기. 

```go
go run server/server.go 
```

를 통해서 서버를 기동하자. 

## Grpc Client 작성하기. 

워크스페이스에서 clients/echoClient.go 파일을 다음과 같이 작성하자. 

```go
package main

import (
	"context"
	"fmt"
	"log"

	"echo-grpc/proto/echopb"

	"google.golang.org/grpc"
)

func main() {
	fmt.Println("Echo Client Luanched.")

	cc, err := grpc.Dial("localhost:9001", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("Could not connect: %v", err)
	}
	
	defer cc.Close()

	c := echopb.NewEchoServiceClient(cc)

	echo := &echopb.EchoRequest{
		Message: "kido",
	}

	result, err := c.Echo(context.Background(), echo)
	if err != nil {
		log.Fatalf("Unexpected error: %v", err)
	}

	fmt.Printf("Result is: %v", result.GetResult())

}
```

- grpc.Dial("localhost:9001", grpc.WithInsecure()): ssl 보안 없이 서버에 접속해서 요청을 한다. 테스트이므로 빠르게 확인하자. 
- echopb.NewEchoServiceClient(cc): echo grpc 서비스를 위한 클라이언트를 생성했다. 
- c.Echo(context.Background(), echo): remote procedure 를 직접 호출한다. 

### 결과 확인하기. 

위 내용은 message 로 'kido' 를 전달하는 클라이언트다. 

서버에서는 Hello <message> 형태로 응답하므로 우리가 원하는 값은 Hello kido 이다. 

```go
go run client/echoClient.go

Echo Client Luanched.
Result is: Hello kido
```

정상적으로 응답이 왔다.

## API Gateway Proxy 생성하기. 

이제는 API Gateway Proxy 코드를 작성해보자. 

워크스페이스에서 gateway/gateway.go 파일을 생성하고 다음과 같이 작성하자. 

```go
package main

import (
	"context"
	"log"
	"net/http"

	"github.com/grpc-ecosystem/grpc-gateway/v2/runtime"

	"echo-grpc/proto/echopb"

	"google.golang.org/grpc"
)

const (
	portNumber = "9000"
	grpcServerPortNumber = "9001"
)

func main() {

	ctx := context.Background()
	mux := runtime.NewServeMux()
	options := []grpc.DialOption{
		grpc.WithInsecure(),
	}

	if err := echopb.RegisterEchoServiceHandlerFromEndpoint(
		ctx, 
		mux,
		"localhost:" + grpcServerPortNumber,
		options,
	); err != nil {
		log.Fatalf("failed to register gRPC gateway: %v", err)
	}

	log.Printf("start HTTP on %s port", portNumber)
	if err := http.ListenAndServe(":" + portNumber, mux); err != nil {
		log.Fatalf("Failed to serve: %s", err)
	}
}
```

- portNumber = "9000": http rest api 의 포트는 9000 이다. 
- grpcServerPortNumber = "9001": grpc 가 떠 있는 포트는 9001 이다. 
- echopb.RegisterEchoServiceHandlerFromEndpoint: rest api 핸들러를 등록한다. 핸들러는 요청이 전달되면, grpcServerPort로 요청을 프록시한다. 
- http.ListenAndServe(: rest api 용 프록시 서버를 기동한다. 

### 테스트하기. 

이제 테스트를 해보자. 

```go
go run gateway/gateway.go 

2021/03/15 07:57:01 start HTTP on 9000 port
```

를 이용하여 프록시 서버를 기동한다. 

이제 curl 요청을 보내보자. 

```go
curl http://localhost:9000/v1/echo/kido

{"result":"Hello kido"}

curl http://localhost:9000/v1/echo/golang

{"result":"Hello golang"}
```

정상적으로 응답이 오는것을 확인할 수 있다. 

## 참고자료

- grpc gateway: https://github.com/grpc-ecosystem/grpc-gateway
- grpc samples: https://devjin-blog.com/golang-grpc-server-3/
