---
layout: post
title:  "JPA기초 설치하기"
date:   2021-02-25 17:00:49 +0900
categories: Java
tags: [Java, JPA, spring, springboot, database, mysql]
toc: true
---

# 프로젝트 세팅하기. 
가장 먼저 수행해야할 작업은 프로젝트 세팅이다. 

## 프로젝트 생성하기. 

스프링은 가장 먼저 [SpringInitializer](https://start.spring.io/) 에 들어가서 프로젝트를 생성한다. 

![setting01](https://user-images.githubusercontent.com/66154381/109122368-d0ab9f80-778b-11eb-8378-9b8d555fd6fe.png)

위와 같이 내용을 입력한다.

- Project: GradleProject  
- Language: Java
- Spring Boot: 2.1.4
- Project Metadata:
    - Group: com.schooldevops.pratical
    - Artifact: simpleboard
    - Name: simpleboard
    - Description: Simple Board Project
    - Package name: 디폴트
    - Packaging: Jar
    - Java: 11
- Dependencies:
    - Lombok: Lombok 은 클래스 파일의 setter/getter 등 기본적으로 필요한 코드를 간단한 어노테이션으로 쉽게 생성할 수 있게 해준다. 
    - Spring Web: web 개발을 할 것이기 때문에 Web 을 선택한다.
    - Spring Data JPA: JPA 를 이용할 것이기 때문에 JPA 를 선택한다. 
    
위와 같이 설치하고 "Generate" 를 클릭하면 프로젝트를 다운로드 받을 수 있다.

## 프로젝트 로드하기. 
이제 IntelliJ 에 프로젝트를 로드한다.

해당 디렉토리로 이동하여 다음 명령을 선택한다. 

```
idea .
```

![setting02](https://user-images.githubusercontent.com/66154381/109122391-de612500-778b-11eb-867e-6789a9067274.png)


위와 같이 오류가 보입니다. Enable 해주면 된다. 

## Lombok 및 어노테이션 세팅하기. 

Lombok 과 Annotation 으로 작업이 되는 JPA 를 위해서 다음 몇가지 설정이 필요하다.

### Lombok 플러그인 설치하기. 

맥에서는 `Cmd + ,` 를 클릭하여 Preference 화면을 오픈한다.

혹은 상단 메뉴에서 Preference 를 선택해도 된다.

그리고 Plugin 을 검색하고, Marketplace 에서 Lombok 을 검색한다. 

![setting03](https://user-images.githubusercontent.com/66154381/109122442-f2a52200-778b-11eb-8a0b-439a28177da0.png)

위와 같이 설치해준다.

### Annotation 활성화 하기. 

![setting04](https://user-images.githubusercontent.com/66154381/109122471-ffc21100-778b-11eb-8523-6375bfab8533.png)

위 상단에 하이라이트 된 것 처럼 `Enable annotation processing` 을 체크한다. 

이제 기본적인 설정이 끝났다.

