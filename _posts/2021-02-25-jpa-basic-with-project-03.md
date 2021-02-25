---
layout: post
title:  "JPA기초 간단한 TODO 생성하기"
date:   2021-02-25 17:06:49 +0900
categories: Java
tags: [Java, JPA, spring, springboot, database, mysql]
toc: true
---

# JPA Basic 으로 간한 TODO 생성하가.

이제는 간단히 TODO 를 작성해 보도록 하겠다. 

## Entity 작성하기. 
단
우선 JPA 는 기존 DB 를 매핑하는 방법과, Entity로 부터 테이블 생성하는 방법을 제공한다. 

우리는 Entity 로 테이을블을 생성해 볼 것이다. 

com.schooldevops.practical.simpleboard.entity.TodoEntity.java 파일을 생성한다. 

```java
package com.schooldevops.practical.simpleboard.entity;

import com.schooldevops.practical.simpleboard.entity.converter.BooleanYNConverter;
import com.schooldevops.practical.simpleboard.entity.converter.LocalDateTimeConverter;
import lombok.*;

import javax.persistence.*;
import java.time.LocalDateTime;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Entity
@Table(name = "Todo")
public class TodoEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column
    private String todo;

    @Column
    @Convert(converter = BooleanYNConverter.class)
    private Boolean done;

    @Column
    @Convert(converter = LocalDateTimeConverter.class)
    private LocalDateTime createdAt;

}

```


우 코드와 같이 우리는 자동생성되는 id, todo로 작업목록, 완료여부, 생성일자를 설정했다. 


- @Entity
    - JPA 에서 @Entity 는 해당 클래스를 테이블과 매핑시키겠다는 의미이며. 
    - 이렇게 테이블과 매핑을 시키면 테이블과 클래스가 연동이 된다. 
- @Table
    - 테이블 이름을 지정할 수 있다. 
    - name 속성으로 테이블 이름을 직접 지정하며, 어노테이션이 생략되면 클래스 이름에 따라 테이블이 생성된다.
- @Id
    - 기본적으로 모든 Entity 는 @Id 선언하는 것이 정석이다. 
    - @Id 는 primary key 로 동작하게 된다. 
- @GeneratedValue(strategy = GenerationType.IDENTITY)
    - Id 를 어떻게 생성할 것인지를 지정하며, 자동으로 생성된 값을 이용하겠다는 의미이다. 
    - strategy 는 어떠한 방법으로 아이디를 생성할 것인지를 지정한다. 
        - GenerationType.AUTO: DB 에 따라 자동으로 설정하겠다는 의미이다. 
        - GenerationType.IDENTITY: MySQL 과 같은 AUTO_INCREMENT 타입의 아이디를 생성하는 방법을 지정한다. 
        - GenerationType.SEQUENCE: Oracle 과 같은 시퀀스 객체를 이용한다.
        - GenerationType.TABLE: 기본키 테이블을 생성하고, 기본키 테이블에서 아이디를 발행하고, 가져가서 사용한다. 
- @Column
    - 칼럼을 나타낸다. 
    - 칼럼의 길이 설정, 테이블의 칼럼이름으로 변경등을 지정할 수 있다. 
- @Convert(converter = LocalDateTimeConverter.class)
    - 특수한 타입의 데이터를 DB 타입으로 변환할 때 사용한다. 
    - 최근 Java 에서 사용하는 LocalDateTime 을 db 의 timestamp 변환할 수 있다. 
    - 사용자가 지정된 컨버터를 이용하면 다양한 형태의 데이터를 변환하여 저장하고, 조회할 수 있다. 

## Converter 작성하기. 
엔터티를 정의할때 @Convert 를 이용했다. 속성값으로 converter 을 만들어서 전달한 것을 확인할 수 있다. 

우리는 2개의 컨버터를 만들 것이다. 우선 테이블에 done 필드의 값이 정상 처리인경우 'Y', 그렇지 않은경우 'N' 으로 저장되도록 컨버터를 작성하자. 

### BooleanYNConverter
com.schooldevops.practical.simpleboard.entity.converter.BooleanYNConverter.java 파일에 다음과 같이 코드를 작성하자. 

```java
package com.schooldevops.practical.simpleboard.entity.converter;

import javax.persistence.AttributeConverter;
import javax.persistence.Converter;

@Converter
public class BooleanYNConverter implements AttributeConverter<Boolean, String> {
    @Override
    public String convertToDatabaseColumn(Boolean attribute) {
        return attribute != null && attribute ? "Y" : "N";
    }

    @Override
    public Boolean convertToEntityAttribute(String dbData) {
        return "Y".equals(dbData);
    }

}

```

- @Converter
    - @Converter 어노테이션을 통해서 혀냊 클래스가 컨버터임을 알려준다. 
- AttributeConverter<Entitytype, TableColumnType> 
    - Converter 은 AttributeConverter 을 상속 받는다. 이때 처음것은 엔터티에서 필드의 타입이며, 두번째는 db 필드에 저장될 타입이다. 
    - 우리는 Boolean 값을 DB 테이블 필드에 string 값으로 'Y', 'N' 으로 저장할 것이라고 정의했다. 
- public String convertToDatabaseColumn(Boolean attribute)
    - 엔터티의 값이 들어오면 DB에 어떤값이 저장될지 변환하는 메소드이다. 
    - 여기서는 Boolean 값이 들어오면, true 라면 'Y', 널이거나 false 인경우 'N' 으로 저장된다. 
- public Boolean convertToEntityAttribute(String dbData)
    - DB 의 값을 엔터티 객체에 담을때 어떻게 변환할지 지정한다. 
    - 여기서는 'Y' 인경우 true, 나머지는 false 로 변환된다. 
    
### LocalDateTimeConverter

LocalDateTime 을 사용하는 것은 Java 개발시에 매우 중요하다. 예전 Date 인해서 발생한 오류와 모호함은 많은 버그와, 장황한 코드를 만들어 낸다. 

LocalDateTime과 LocalDate 은 이러한 장황함을 제거하고, 오류가 적은 코드를 만들 수 있다. 그리고 많은 편의 기능은 덤이다.

com.schooldevops.practical.simpleboard.entity.converter.LocalDateTimeConverter.java 파일을 생성하고, 다음과 같이 작성하자. 

```java
package com.schooldevops.practical.simpleboard.entity.converter;

import javax.persistence.AttributeConverter;
import javax.persistence.Converter;
import java.sql.Timestamp;
import java.time.LocalDateTime;

@Converter
public class LocalDateTimeConverter implements AttributeConverter<LocalDateTime, Timestamp> {
    @Override
    public Timestamp convertToDatabaseColumn(LocalDateTime attribute) {
        return attribute == null ? null : Timestamp.valueOf(attribute);
    }

    @Override
    public LocalDateTime convertToEntityAttribute(Timestamp dbData) {
        return dbData == null ? null : dbData.toLocalDateTime();
    }
}

```

위 부분은 더 설명하지 않아도 충분히 설명적이다. 

LocalDateTime 을 java.sql.Timestamp 로 변환한다. 

## 프로그램 실행하기. 

정상으로 테이블이 생성되는지 실험해 보자. 

JPA 로그를 확인하여 테이블 생성되는 과정을 확인해보기 위해서 application.yml 에 다음 내용을 추가하자. 

```java
  jpa:
    properties:
      hibernate:
        show_sql: true
        use_sql_comment: true
        format_sql: true
    hibernate:
      ddl-auto: create

logging:
  level:
    org.hibernate.type: trace
```

- spring.jpa.properties.hibernate.show_sql=true 
    - 쿼리를 로그로 확인할 수 있다. 
- spring.jpa.properties.hibernate.use_sql_comment=true
    - 쿼리 파라미터를 로그로 확인할 수 있다. 
- spring.jpa.properties.hibernate.format_sql=true
    - 쿼리를 보기좋게 포맷팅한다. 
- spring.jpa.hibernate.ddl-auto=create
    - none: 기본값으로 아무작업도 하지 않는다. (이미 존재하는 DB 테이블을 이용할때 주로 사용한다.)
    - create: 매번 DDL 을 수행한다. (초기 환경 설정할때 이용하며, 실제 사용하면 큰일이 발생한다. 해보지 말길)
    - update: 변경되는 사항이 있을 경우에만 ddl 수정사항이 발생한다.    
    - create-drop: SessionFactory 가 생성될때 drop and create 수행하여 테이블을 생성하고, SessionFactory가 종료될때 drop 처리된다.
    - validate: 변경되는 사항이 있다면 변경사항을 노출하고 어플리케이션을 종료한다. (즉, Entity와 Table이 실제 매핑이 맞지 않으면, 어플리케이션이 실행되지 않으므로, 변경사항에 대한 문제를 찾을때 유용하며, 실제 운영에 주로 사용한다.)
- logging.level.org.hibernate.type=trace
    - 로그가 노출 되도록 한다. 

SimpleboardApplication.java 파일을 실행해 보자. 

로그를 확인해보면 다음과 같은 로그를 확인할 수 있다. 

```java
Hibernate: 
    
    drop table if exists todo
Hibernate: 
    
    create table todo (
       id bigint not null auto_increment,
        created_at datetime(6),
        done varchar(255),
        todo varchar(255),
        primary key (id)
    ) engine=InnoDB
```

정상으로 테이블이 생성 되었음을 알 수 있다. 

## Repository 작성하기. 
엔터티를 만들었다면 이제는 리포지토리를 생성하고, 조회할 수 있도록 하자. 

JPA 에서는 기본적으로 Repository 인터페이스가 있고 이를 상속 받은것이 CrudRepository 이다. 

CrudRepository 는 기본적은 CRUD 를 위한 리포지토리이며 이를 상속 받은 것이 PagingAndSortingRepository 가 있다. 

PagingAndSortingRepository 는 이름이 알려주듯이 기본 CRUD 기능 위에 페이징과 소팅을 지원한다. 

마지막으로 JPARepository 가 있으며 JPARepository 는 PagingAndSortingRepository 를 상속 받는다. 

이 말은 JPARepsitory 는 가장 많은 기능을 가지고 있으며, 좀더 디테일한 기능들을 가지고 있다. flush 와 같은 메소드를 지원한다. 

우리는 여기서는 간단한 쿼리 이므로 CrudRepository 를 이용해서 작업할 것이다.
  
com.schooldevops.practical.simpleboard.repository.TodoRepository.java 파일을 생성하고 다음과 같이 작성하자. 

```java
package com.schooldevops.practical.simpleboard.repository;

import com.schooldevops.practical.simpleboard.entity.TodoEntity;
import org.springframework.data.repository.CrudRepository;

import java.util.List;

public interface TodoRepository extends CrudRepository<TodoEntity, Long> {

    List<TodoEntity> findAll();
}

```

별다른 내용이 없다. 단지 인터페이스이며, JPA 가 위 이터페이스 구현을 자동을 처리해준다. 

## CRUD REST API 작성하기. 

이제는 컨트롤러를 만들고 Todo list 를 CRUD 해보자. 

```java
package com.schooldevops.practical.simpleboard.controller;

import com.schooldevops.practical.simpleboard.entity.TodoEntity;
import com.schooldevops.practical.simpleboard.repository.TodoRepository;
import org.springframework.web.bind.annotation.*;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

@RequestMapping("/todos")
@RestController
public class TodoController {

    final TodoRepository todoRepository;

    public TodoController(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    @GetMapping(path = "")
    public List<TodoEntity> getAllTodos() {
        return todoRepository.findAll();
    }

    @PostMapping
    public TodoEntity postTodo(@RequestBody TodoEntity todo) {
        if (todo.getCreatedAt() == null) {
            todo.setCreatedAt(LocalDateTime.now());
        }
        return todoRepository.save(todo);
    }

    @PutMapping(path = "/{id}")
    public TodoEntity putTodo(@PathVariable("id") Long id, @RequestBody TodoEntity todo) {
        Optional<TodoEntity> todoResult = todoRepository.findById(id);
        TodoEntity todoEntity = todoResult.orElseThrow(() -> new RuntimeException("There are any result by " + id));

        if (todo.getTodo() != null) {
            todoEntity.setTodo(todo.getTodo());
        }

        if (todo.getDone() != null) {
            todoEntity.setDone(todo.getDone());
        }

        return todoRepository.save(todoEntity);
    }

    @DeleteMapping(path = "/{id}")
    public TodoEntity deleteTodo(@PathVariable("id") Long id) {
        Optional<TodoEntity> todoResult = todoRepository.findById(id);
        TodoEntity todoEntity = todoResult.orElseThrow(() -> new RuntimeException("There are any result by " + id));

        todoRepository.delete(todoEntity);

        return todoEntity;
    }
}

```

위처럼 작성했다. 

이제 Curl 이나 Http-Client 플러그인 등을 이용하여 CRUD 테스트를 진행할 수 있다. 

여기서는 Http-Client 를 이용해서 생성했다. 

resources/http-test/todoTest.http
```java
###
POST localhost:8080/todos
Content-Type: application/json

{
  "todo": "Get Up Early",
  "done": false
}

###
PUT localhost:8080/todos/1
Content-Type: application/json

{
  "todo": "Go to bed",
  "done": true
}

###
GET localhost:8080/todos

###
DELETE localhost:8080/todos/2
```

지금까지 간단하게 CRUD 를 작성해 보았다. 

컨버터를 이용한 하나의 테이블에 CRUD 를 생성하면서 JPA 의 기본 동작을 이해 할 수 있게 되었다. 
