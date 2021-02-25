---
layout: post
title:  "JPA기초 DB 세팅하기"
date:   2021-02-25 17:05:49 +0900
categories: Java
tags: [Java, JPA, spring, springboot, database, mysql]
toc: true
---

# DB 세팅하기. 

JPA 를 이용할 것이기 때문에 DB 세팅이 필요하. 

우리는 MariaDB 를 이용하여 데이터소스를 생성해 보겠다. 

## MariaDB 설치하기. 

MariaDB 는 설치 파일을 다운로드 받아서 설치할 수 있고, 혹은 Docker 로 컨테이너를 실행할 수 있다. 

Docker은 [다음 경로](https://docs.docker.com/desktop/)에서 다운로드 받는다.

자신의 환경에 맞게 설치해 주시면 된다.

## MariaDB Container 실행하기. 

우리는 MariadB 컨테이너를 Docker 에 올려서 실행할 것이다. 그리고 테스트 환경은 Mac 에서 수행할 것이다. 

다음 명령을 실행한다. 
 
```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password123 --name mariadb-local -v /Users/baekido/data/mariadb:/var/lib/mysql mariadb:latest 
``` 

위 명령으로 MariaDB 컨테이너를 실행하기 . 

- docker run
    - docker 를 실행합니다. 로컬에 이미지가 없다면 다운로드 받는다. 
- -d
    - detach 모드이며, 백그라운드로 실행되도록 한다.
- -p 3306:3306
    - 외부 포트를 컨테이너 내부 포트와 연결한다. <호스트포트>:<컨테이너포트> 
    - 이렇게 하면 localhost:3306 으로 접근할 수 있다. 
- -e MYSQL_ROOT_PASSWORD=...
    - -e는 환경변수를 전달합니다. 우리는 루트 패스워드를 전달해 주었다. 
- --name
    - --name 은 컨테이너 이름을 지정한다. 생략하면 Docker 가 자동으로 컨테이너 이름을 생성한다. 
- -v
    - -v는 호스트의 볼륨과, 컨테이너 볼륨을 연결한다. 
    - 이렇게 하면 mariadb 가 컨테이너의 볼륨에 쓰기를 하면, 호스트의 /Users/baekido/data/mariadb 라는 위치에 쓰기가 된다. 
- mariadb:latest
    - 이미지의 이름:태그 를 지정하여 실제 이미지를 컨테이너로 실행한다.
    - tag 값을 latest 는 가장 최신의 버젼을 의미하며, 사용자가 직접 버젼을 지정할 수 있다. 

### 컨테이너 상태 확인하기. 

```
docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ab119ffdbf64        mariadb:latest      "docker-entrypoint.s…"   6 minutes ago       Up 6 minutes        0.0.0.0:3306->3306/tcp   mariadb-local

```

위와 같이 컨테이너 상태를 확인할 수 있다.

### 클라이언트를 통해서 접속하기. 

MariaDB 에 접근하는 클라이언트는 [MySQLWorkbench](https://dev.mysql.com/downloads/workbench/) 이나 [DBeaver](https://dbeaver.io/download/) 에서 다운로드 받아서 사용하면 된다

### 데이터베이스 생성 및 사용자 생성 권한 주기. 

- 사용자생성
```
CREATE USER 'boarduser'@'%';
ALTER USER 'boarduser'@'%' IDENTIFIED BY 'boarduser123' ;
```

- 데이터베이스생성하기
```
CREATE DATABASE `simpleboard`;
```

- 권한부여하기 
```
GRANT Alter ON simpleboard.* TO 'boarduser'@'%';
GRANT Create ON simpleboard.* TO 'boarduser'@'%';
GRANT Create view ON simpleboard.* TO 'boarduser'@'%';
GRANT Delete ON simpleboard.* TO 'boarduser'@'%';
GRANT Delete history ON simpleboard.* TO 'boarduser'@'%';
GRANT Drop ON simpleboard.* TO 'boarduser'@'%';
GRANT Grant option ON simpleboard.* TO 'boarduser'@'%';
GRANT Index ON simpleboard.* TO 'boarduser'@'%';
GRANT Insert ON simpleboard.* TO 'boarduser'@'%';
GRANT References ON simpleboard.* TO 'boarduser'@'%';
GRANT Select ON simpleboard.* TO 'boarduser'@'%';
GRANT Show view ON simpleboard.* TO 'boarduser'@'%';
GRANT Trigger ON simpleboard.* TO 'boarduser'@'%';
GRANT Update ON simpleboard.* TO 'boarduser'@'%';
```

## Spring DataSource 연동하기. 

### mariadb connector 연동하기. 

[MavenRepo](https://mvnrepository.com/artifact/org.mariadb.jdbc/mariadb-java-client/2.6.0) 에서 mariadb java client 드라이버를 검색한다.

의존성을 build.gradle 에 추가하자.

그리고 HikariCP 로 DB 커넥션 풀을 위한 의존성도 아래 추가한다. 

```
	// https://mvnrepository.com/artifact/org.mariadb.jdbc/mariadb-java-client
	compile group: 'org.mariadb.jdbc', name: 'mariadb-java-client', version: '2.6.0'
	// https://mvnrepository.com/artifact/com.zaxxer/HikariCP
	compile group: 'com.zaxxer', name: 'HikariCP', version: '3.4.5'
``` 

### application.yml 에 설정 추가하기. 

resources/application.yml 에 다음 내용을 추가한다. 

```
spring:
  datasource:
    username: boarduser
    password: boarduser123
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mysql://localhost:3306/simpleboard?serverTimeZone=Asia/Seoul&autoReconnect=true
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
      pool-name: board-pool
      maximum-pool-size: 20
      max-lifetime: 1800000
      idle-timeout: 30000
```

데이터데이스에 연동을 위한 설정 부분과, hikari connection pool 에 대한 설정을 위와 같이 추가한다. 

지금까지 간단하게 DB 연동을 살펴 보았다.

