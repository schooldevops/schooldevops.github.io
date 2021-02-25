---
layout: post
title:  "JPA기초 ManyToMany"
date:   2021-02-25 17:10:49 +0900
categories: Java
tags: [Java, JPA, spring, springboot, database, mysql]
toc: true
---

# ManyToMany

관계형 데이터베이스를 이용하다보면 ManyToMany 관계를 종종 만들어야 할 때가 있다. 

## 시나리오. 

- 사용자 정보를 담고 있는 User 가 있다. 
- 시스템의 다양한 사용정보를 정의한 Role 이 있다. 
- 사용자는 다양한 Role 을 가질 수 있다. 
- Role 은 여러 사용자에 할당 될 수 있다. 

이러한 시나리오가 전형적인 M:N 관계이다. 이러한 관계는 연관 테이블을 생성하여 비즈니스 니즈를 해결하는 것이 일반적이다. 

<img width="502" alt="M_n_Relation" src="https://user-images.githubusercontent.com/66154381/109124123-e9b55000-778d-11eb-8ade-ae89d2595fad.png">

위 다이어그램에서 보는 바와 같이 user_id, role_id 를 각각 가진 user_role 이라는 테이블으 생성 되었고. 

user : user_role = 1 : N 관계가 형성이 된다. 

role : user_role = 1 : N 관계가 형상이 된다. 

이렇게 중간 테이블을 만들어서 연관을 맺어주면 쉽게 M:N 관계를 설정할 수 있다. 

## Role 엔터티 생성하기. 

이제 Role 엔터티를 생성해보자. Role 은 제목만 가지고 있고, Role 에 대한 상세 권한은 이번 내용에 포함시키지 않았다. 

단지 사용자가 어떠한 Role 을 가질지만으로 M:N 관계를 설저해볼 것이다. 

User.java 엔터티를 생성하고 다음과 같이 작성한다. 

```java
package com.schooldevops.practical.simpleboard.entity;

import lombok.*;

import javax.persistence.*;
import java.util.Set;

@AllArgsConstructor
@NoArgsConstructor
@Builder
@Getter
@Setter
@Entity
@Table(name = "role")
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

}

```

위 내용은 단순히 id(자동생성) 과 role 이름만 있는 단순한 엔터티를 만들었다. 

## 연관 맺어주기. 

이제부터 User 와 Role 에 대해서 연관을 맺어줄 것이다. 

Role.java 부터 연관을 작성해보자. 

```java
... 중간생략 ...
    @ManyToMany
    @JoinTable(
            name="user_role",
            joinColumns = @JoinColumn(name = "role_id"),
            inverseJoinColumns = @JoinColumn(name = "user_id")
    )
    private Set<User> users;
```

위 내용과 같이 연관을 생성했다. 

- 하나의 Role 은 여러 User 를 가질 수 있다. 
- @ManyToMany: M:N 관계를 위한 어노테이션을 지정했다. 
- @JoinTable: 조금전에 이야기한 연관 테이블 속성을 정의한다. 
    - name="user_role": user_role 라는 테이블을 생성하라고 지시한다. 
    - joinColumns: 이것은 Role 입장에서 자신의 키가 user_role 테이블에 어떻게 매핑될지 설정한다. 
        - @JoinColumn(name = "role_id"): user_role 테이블에 role_id 라는 필드로 매핑됨을 미한다. 
    - inverseJoinColumns: 이것은 반대편 엔터티를 의미하며 Role 입장에서는 User 가 된다. 
        - @JoinColumn(name = "user_id"): user_role 테이블에 user 테이블의 id 가 매핑되며 필드 이름은 user_id 가 된다. 
 
위와 같이 연관을 매핑해 보았다. 

User.java 테이블 연관 맺어주기. 

```java

... 중간 생략 ...
    @ManyToMany
    @JoinTable(
            name = "user_role",
            joinColumns = @JoinColumn(name = "user_id"),
            inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles;

    public void addRole(Role role) {
        if (roles == null) {
            roles = new HashSet<>();
        }

        roles.add(role);
    }
... 중간 생략 ...
```

Role에 매핑을 걸어보았으니, 이 부분은 그리 어렵지 않다. 

- @ManyToMany: 역시 동일하게 M:N 관계를 선언하는 어노테이션이다. 
- @JoinTable: 연관 테이블을 정의해준다. 
    - name = "user_role": user_role 테이블을 생성하도록 지시한다.
    - joinColumns: User 입장에서 자신의 키가 연관테이블의 어떠한 필드와 매핑되는지 알려준다. 
        - @JoinColumn(name = "user_id"): user_id 이름으로 매핑되도록 설정했다. 
    - inverseJoinColumns: User의 반대편 엔터티이므로 Role 를 의미한다. 
        - @JoinColumn(name = "role_id"): 연관 테이블에 role_id 필드가 role 키와 매핑된다. 

그리고 User를 기준으로 Role 을 매핑해 줄 것이므로, addRole 처럼 편의 메소드를 만들어 주었다. 

## 생성된 결과 확인하기. 

이제 위 연관을 이용하여 생성된 테이블 쿼리를 확인해 볼 차례이다. 

```java
Hibernate: 
    
    create table role (
       id bigint not null auto_increment,
        name varchar(255) not null,
        primary key (id)
    ) engine=InnoDB
```

role 테이블이 생성 되었다. 기본키는 id 이다. 

```java
Hibernate: 
    
    create table user (
       id varchar(255) not null,
        birth varchar(255),
        created_at datetime(6),
        name varchar(255),
        primary key (id)
    ) engine=InnoDB
```

user 테이블도 생성이 되었으며 역시 기본키는 id 이다. 

```java
Hibernate: 
    
    alter table user_role 
       add constraint FKa68196081fvovjhkek5m97n3y 
       foreign key (role_id) 
       references role (id)
```

user_role 테이블이 생성 되었다. 이것은 우리가 의도한 테이블이다. 

그리고 다음과 같이 양방향 Foreign Key 가 생성되었다. 

```java
Hibernate: 
    
    alter table user_role 
       add constraint FKa68196081fvovjhkek5m97n3y 
       foreign key (role_id) 
       references role (id)
Hibernate: 
    
    alter table user_role 
       add constraint FK859n2jvi8ivhui0rl0esws6o 
       foreign key (user_id) 
       references user (id)
```

보는 바와 같이 user_role 테이블에 role_id 는 role 테이블의 id 기본키와 foreign key 관계를 가진다. 

역시 user_role 테이블에 user_id 는 user 테이블의 id 기본키와 foreign key 관계를 가지도록 생성 되었다.

처음 우리가 지정한 방식대로 ManyToMany 가 생성된 것을 확인할 수 있다. 


 