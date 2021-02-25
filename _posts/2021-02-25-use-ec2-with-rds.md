---
layout: post
title:  "AWS Go WebProgram 과 RDS 연동 하기 (무대뽀 버젼)"
date:   2021-02-25 18:50:49 +0900
categories: Go
tags: [mysql, Go, golang, gorilla, rds, aws, ec2]
toc: true
---

# 무식하게 EC2와 RDS 연동하여 Go Web Programming 서비스하기

AWS 를 이용하면 일반적으로 Web 프로그램을 EC2 에 런치하여 서비스 하고, 뒷단 데이터베이스는 RDS 를 이용하여 개발을 한다. 

이제 간단한 웹 프로그램을 통해서 어떻게 EC2와 RDS를 생성하고 연동하는지 알아보자. 

우리가 사용할 소스 코드는 다음 위치에서 찾을 수 있다. [todo-server](https://github.com/schooldevops/go_todo_server) 

## RDS 설치하기. 

AWS RDS는 Relation Database Service 이며, 다양한 관계형 데이터베이스 엔진을 제공하여, 쉽고 빠르게 데이터베이스를 생성하고 운영할 수 있도록 해준다. 

![rds01](https://user-images.githubusercontent.com/66154381/109129515-11a7b200-7794-11eb-8d83-23bbec751a23.png)

- 표준 생성: 사용자가 직접 설정할 수 있도록 합니다. 
- 엔진 옵션: 우리가 사용할 엔딘은 MySQL 이다. 그러니 MySQL 을 선택하자. 
- 버젼: MySQL 8.0.20 을 선택하자. (현재 최신버젼을 선택한다.)
- 템플릿: 템플릿은 실제 제품 개발이 아니라 테스트를 해보려면 프리티어를 설치하자. 보통 개발시에는 (개발/테스트) 를 선택하면 된다. 

![rds02](https://user-images.githubusercontent.com/66154381/109129571-1f5d3780-7794-11eb-8076-1b9d5b63c462.png)

- DB 인스턴스 식별자: 리젼당 고유한 이름이 될 수 있도록 이름을 작성하자. 필자는 go-todo-kido-db 라고 작성했다. 
- 마스터 사용자 이름: DB 에 접근할 접근 계정이다. boarduser 이다. 
- 마스터 암호 / 암호 확인: 적당한 암호를 입력하자. 
- DB 인스턴스 클래스: 프리티어를 선택했기 때문에 db.t2.micro 가 생성이 된다. 이는 테스트 용도이므로 실제 개발이나, 운영에서는 성능이 더욱 좋은 인스턴스 클래스를 이용해야한다. 
    - 1 vCPU / 1 GiB RAM / EBS 옵션 사용 못함이 기본이다. 

![rds03](https://user-images.githubusercontent.com/66154381/109129642-33a13480-7794-11eb-8538-50440aa4515a.png)

- 스토리지 유형: DB에 사용할 스토리지 를 지정한다. 일반적인 범용 SSD를 선택한다. 
- 할당된 스토리지: 최소 20GiB 이상이어야 IOPS 성능 개선을 이룰수 있다고 한다. 우린 테스트이니 10으로 잡았다. 
- 스토리지 자동 조정: 필요한경우 스토리지를 자동으로 늘려주는 옵션이다. (기본적으로 선택하는게 좋다.)
- 최대 스토리지 임계값: 스토리지를 늘일 임계값을 지정한다. 기본으로 그대로 두었다. 
- 다중 AZ 배포: HA를 향상시키기 위해서 다중 AZ를 배포하는 것이 좋다. (여러 AZ에 DB가 설치되므로 안정적인 가용성을 얻을 수 있다. ) 프리티어에서는 선택할 수 없다. 

![rds04](https://user-images.githubusercontent.com/66154381/109137584-b4fcc500-779c-11eb-8e94-a3e9550f675a.png)

- Virtual Private Cloud(VPC): 설치될 VPC를 지정한다. 
- 서브넷 그룹: VPC내부에 서브넷 그룹을 지정한다. 
- 퍼블릭 엑세스 가능: 필자의 경우에는 "예"를 선택했다. 로컬로 접근하여 DB스키마와 툴을 통해서 직접 컨트롤 하고자 이렇게 설정했다. 
- VPC 보안그룹: 새로 생성을 해 주었다. DB 인스턴스에 접근가능한 보안그룹이 이미 있다면 기존항목을, 아닌경우 생성하면서 자동으로 보안그룹을 생성한다. 
    - 퍼블릭 액세스 가능이기 때문에 보안그룹은 3306에 대해서 자신의 IP만 접근하도록 보안그룹이 설정될 것이다. 
- 새 VPC 보안 그룹 이름: 보안그룹 이름을 지정한다. 
- 가용영역: 어떤 가용영역에 DB를 설치할지 결정한다. 
- 데이터베이스 포트: 기본값인 3306을 둔다. 실제 운영을 할때에는 포트를 다르게 사용하는 것을 권한다. 

![rds05](https://user-images.githubusercontent.com/66154381/109129737-503d6c80-7794-11eb-8f70-fda53305c272.png)

- 데이터베이스 인증 옵션: 여기서는 암호로 접속할 수도 있고, IAM 을 통해서 접근가능하도록 설정할 수 있게 했다. 

![rds06](https://user-images.githubusercontent.com/66154381/109129787-5d5a5b80-7794-11eb-9c09-22bdf4f00a52.png)

- 초기 데이터베이스 이름: simpleboard 를 넣었다. 
- DB파라미터 그룹: DB의 config 정보를 지정하는 것으로 특별히 바꿀것이 없다면 기본값을 이용한다. 
- 옵션 그룹: 옵션 그룹도 동일하게 기본값을 지정했다. 
- 자동백업 활성화: 데이터 베이스의 특정 시점 데이터를 스냅샷으로 저장한다. 
- 백업 보존 기간: 얼마나 스냅샷을 저장할지 지정하는 것으로 기본값인 7일을 그대로 유지했다. 
- 백업 기간: 자동 백업할 기간을 지정하는 것으로 기본값으로 설정없음을 했다. 
- 나머지모두 기본값을 이용한다. 

![rds07](https://user-images.githubusercontent.com/66154381/109129843-69deb400-7794-11eb-83a9-fa7ff2faa6b8.png)

- 마이너 버젼 자동 업그레이드 사용: 마이너 업그레이드는 실행에 영향을 주지 않으므로, 자동업그레이드 설정했다. 
- 나머지모두 기본값을 이용한다. 

이후 "데이터베이스 생성" 을 클릭하자. 

![rds08](https://user-images.githubusercontent.com/66154381/109137730-e2e20980-779c-11eb-8724-f1ce36d22e35.png)

- 생성중으로 데이터베이스가 생성되고 있다. 
- 데이터베이스는 어느정도 생성 시간이 걸리므로 느긋하게 기다리자. 

![rds09](https://user-images.githubusercontent.com/66154381/109137785-f7260680-779c-11eb-9a60-f3af4183b138.png)

- 생성된 데이터베이스가 생성이 된경우 위와 같이 노출된다. 

![rds10](https://user-images.githubusercontent.com/66154381/109130000-9692cb80-7794-11eb-814c-a906b2aee77c.png)

생성된 정보를 상세하게 확인할 수 있다. 


## 쿨라이언트에서 접근하기. :

데이터베이스를 외부에서 접근할 수 있도록 보안 그룹이 지정되었으므로, 자신의 PC에서 접근이 가능하다. 

적절한 DBMS 툴을 이용하여 연결해보면 정상적으로 접근됨을 알 수 있다. 

DB에 접근하여 다음 쿼리를 실행하자. 

```sql

drop table alarm_target;
drop table todo_alarm;
drop table todos;

create table todos (
    id bigint(20) not null auto_increment comment 'id',
    user_id bigint(20) null comment '사용자 아이디',
    title varchar(500) not null comment '제목',
    priority varchar(20) not null comment '우선순위: T, A, B, C, D',
    status varchar(20) not null comment '상태: REGISTED, DOING, DONE, HOLD, ARCHIVE',
    completion_level int null comment '완성도: 0 ~ 100 %',
    created_at timestamp not null comment '생성일',
    modified_at timestamp null comment '수정일',
    done_at timestamp null comment '완료일',
    primary key (id)
)
comment 'TODO 목록';

create table todo_alarm (
    id bigint(20) not null auto_increment comment 'id',
    todo_id bigint(20) not null comment 'todo id',
    period_type varchar(20) not null comment 'alarm type: ONCE, INTERVAL',
    alarm_type varchar(20) not null comment 'PUSH, MAIL',
    alarm_date date null comment '알람일자, 특정일',
    alarm_time time not null comment '알람시간',
    active_yn char(1) not null default 'Y' comment '활성화여부',
    created_at timestamp not null comment '생성일',
    modified_at timestamp null comment '수정일',
    primary key (id)
)
comment 'ALARM';

alter table todo_alarm add constraint fk_with_todos foreign key(todo_id) references todos (id) ON DELETE CASCADE;

create table alarm_target (
    id bigint(20) not null auto_increment comment 'id',
    todo_alarm_id bigint(20) not null comment '알람 아이디',
    phone varchar(100) null comment '전화번호',
    email varchar(100) null comment '메일주소',
    user_id bigint(20) null comment '사용자 아이디',
    active_yn char(1) not null default 'Y' comment '활성화여부',
    created_at timestamp not null comment '생성일',
    modified_at timestamp null comment '수정일',
    primary key (id)
)
comment '알람대상정보';

alter table alarm_target add constraint fk_with_alarm_target foreign key(todo_alarm_id) references todo_alarm(id) ON DELETE CASCADE;
```

## EC2 설치하기. 

이제 DB 스키마를 완성했으니 EC2를 생성할 차례이다. EC2 > 인스턴스 생성을 클락하자. 

![ec2_01](https://user-images.githubusercontent.com/66154381/109130513-29cc0100-7795-11eb-993d-cb7fbc84fd74.png)

- Amazon Linux 2 AMI 를 선택했다. 

![ec2_02](https://user-images.githubusercontent.com/66154381/109130546-33edff80-7795-11eb-9394-c5d534fe8c84.png)

- 테스트이므로 t2.micro 를 선택했다. 

![ec2_03](https://user-images.githubusercontent.com/66154381/109130585-40725800-7795-11eb-8588-1d46770e10ab.png)

- 인스턴스 세부 구성에서 vpc 를 적절히 선택하자. DB와 동일한 VPC 를 선택해야함을 꼭 확인하자. 

![ec2_04](https://user-images.githubusercontent.com/66154381/109130624-4bc58380-7795-11eb-8481-e01a76f7a0cf.png)

- 태그를 작성해서 인스턴스를 식별할 수 있도록 하자. 

![ec2_05](https://user-images.githubusercontent.com/66154381/109130670-57b14580-7795-11eb-9c07-d3df83556dd3.png)

- 새 보안그룹을 생성하고, 이름을 gotodo-web-sg 를 작성하자. 
- ssh: ssh 접속을 허용하도록 한다. 
- 9999: 9999포트를 열어서 외부에서 EC2로 9999 포트로 접근할 수 있도록 설정했다. 

![ec2_06](https://user-images.githubusercontent.com/66154381/109130709-64ce3480-7795-11eb-8072-9907b0aca076.png)

- 생성된 보안 그룹은 위와 같다. 

## EC2 환경 설정하기. 

이제 EC2가 생성이 되었으므로 인스턴스에 ssh 로 접근하자. 

그리고 아래 과정대로 Docker와 Git을 설치하고 프로그램을 실행한다. 

### Docker 설치 

아래 커맨드로 docker 을 설치하고 권한을 주자. 
그리고 docker-comose 가 필요하다면 아래 내용처럼 docker-compose를 설치도 해주면 된다. 

```shell
amazon-linux-extras install docker
service docker start
usermod -a -G docker ec2-user

curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### Git 설치

```shell
yum -y install git 

git clone https://github.com/schooldevops/go_todo_server.git
cd go_todo_server
```

소스코드 역시 가져왔다. 

소스 코드에서 중요 부분은 mysql.go 내보에 있는 다음 부분이다. 

```go
var connectionString = "boarduser:boarduser123@tcp(go-todo-kido-db.cawucurfyhe9.ap-northeast-2.rds.amazonaws.com:3306)/simpleboard?charset=utf8&parseTime=True&loc=Local"

```

go-todo-kido-db.cawucurfyhe9.ap-northeast-2.rds.amazonaws.com:3306 은 DB의 public domain 이름이다. 이 도메인 이름은 HA 가 구성되어 마스터 서버가 죽고, 슬레이브가 마스터로 변경이 되어 DBMS가 시작되어도 변경되지 않는다. 

### Docker Build

Docker 를 빌드하고 실행하자. 

```shell
docker build -t go_todo:v1.0 .
```

### Docker Run 및 테스트하기 

```shell
docker run -d -p 9999:9999 --name go_todo go_todo:v1.0
```

위와 같이 컨테이너를 실행했다. 

이제 curl로 다음 코드를 실행하자. 

```shell
curl -i http://ec2-3-36-97-155.ap-northeast-2.compute.amazonaws.com:9999/api/todos 
```

결과가 어떻게 되었을까? 

Connection Timeout 가 발생하면서 접속이 안될 것이다. 

이제 EC2가 RDS 에 접근할 수 있도록 RDS의 보안 그룹을 설정해 주어야한다. 

## EC2가 RDS 를 이용하도록 SecurityGroup 설정해주기. 

![rds_ec2_01](https://user-images.githubusercontent.com/66154381/109130865-8c250180-7795-11eb-9f64-ed223e95ab3f.png)

위 그림과 같이 RDS의 보안 그룹에 인바운드를 선택하고, 편집을 클릭한다. 

추가를 선택하고 다음과 같이 작성해준다. 

- 프로토콜: TCP
- 포트 범위: 3306
- 소스: 소스 부분에 EC의 Security group 을 작성해주자. 

### 테스트하기. 

이제 테스트를 해 보면 요청에 대한 응답값이 정상으로 반환됨을 확인할 수 있다. 

## 결론

지금까지 EC2와 RDS를 연동하는 방법을 하나하나 알아 보았다. 

핵심은 RDS의 보안 그룹에 EC2 --> RDS로 인바운드 할 수 있도록 보안그룹 규칙을 지정해 줌으로써 EC2가 RDS에 접근할 수 있게 되었다. 

지금까지 무식하게 EC2에 웹 서버를 올려 보았다. 

- 서버가 중지 되었다가 재 실행될때 Go Web Program 이 동작하지 않는 문제도 생길 것이고
- 직접 사용자가 콘솔로 들어가서 소스를 pull 하고, 컨테이너를 만들어 실행하는 방법으로 접근하는 것은 비효울 적이다. 

위와 같은 부분도 함께 고민해서 안정적인 서비스를 만들 필요가 있다. 

