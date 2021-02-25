---
layout: post
title:  "JPA기초 OneToOne Bidirection"
date:   2021-02-25 17:08:49 +0900
categories: Java
tags: [Java, JPA, spring, springboot, database, mysql]
toc: true
---

# OneToOne Bidirection 매핑

OneToOne Unidirection 매핑을 지금까지 해보았다. 

- UniDirection
    - User가 UserDetail 를 포함하고 있는구조. 
    - User 에서 UserDetail 을 참조할 수 있다. 
    - UserDetail 에서 User를 접근할 수 없음
    - DB 조회시 User 를 조회하고, 필요시 UserDetail 을 Lazy Loading 해서 조회할 수 있다. 
- Bidirection
    - User가 UserDetail 를 포함하는 구조. 
    - UserDetail 이 User 을 포함하는 구조.
    - User 에서 UserDetail 을 참조할 수 있다.
    - UserDetail에서 User 을 참조할 수 있다. 
    - DB 조회시 User 를 조회하고, 필요시 UserDetail 을 Lazy Loading 해서 조회할 수 있다.
    - DB 조회시 UserDetail 을 조회하고, 필요시 User을 Lazy Loading 해서 조회할 수 있다. 
    
위와 같이 Bidirection 연관을 맺어주면 어느쪽으로 조회하든 편리하게 필요한 정보를 함께 가져올 수 있다. 

이제 Bidirection 연관을 맺어보자. 

## User 연관 설정

User.java 를 다음과 같이 작성하자. 

```java
package com.schooldevops.practical.simpleboard.entity;

import com.schooldevops.practical.simpleboard.dto.UserDto;
import com.schooldevops.practical.simpleboard.entity.converter.LocalDateTimeConverter;
import lombok.*;

import javax.persistence.*;
import java.time.LocalDateTime;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Entity
@Table(name = "user")
public class User {

    @Id
    @Column(name = "id", unique = true)
    private String id;

    @Column
    private String name;

    @Column
    private String birth;

    @Column
    @Convert(converter = LocalDateTimeConverter.class)
    private LocalDateTime createdAt;

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL, orphanRemoval = true)
    private UserDetail userDetail;

    public void setUserDetail(UserDetail userDetail) {
        this.userDetail = userDetail;
        userDetail.setUser(this);
    }

    @Transient
    public UserDto getDTO() {
        UserDto userDTO = UserDto.builder()
                .id(this.id)
                .name(this.name)
                .birth(this.birth)
                .createdAt(this.createdAt)
                .build();

        if (userDetail != null) {
            userDTO.setUserDetail(userDetail.getDTO());
        }
        return userDTO;
    }
}

```

위 내용중 핵심은 @OneToOne 어노테이션 부분이다. 

User 객체에 UserDetail 을 포함하도록 하였고. 

- @OneToOne: 명시적으로 User와 UserDetail 은 1:1임을 선언했다.
- cascade=CascadeType.ALL: User의 오퍼레이션이 UserDetail 에 전파되는 것을 전부다로 설정했다. 
- orphanRemoval=true: 부모인 User가 삭제되면 자식인 UserDetail 역시 함께 삭제된다. 

그리고 다음으로 편의 메소드를 작성했다. 

```java
    public void setUserDetail(UserDetail userDetail) {
        this.userDetail = userDetail;
        userDetail.setUser(this);
    }
``` 

이것은 userDetail 을 User에 설정할때 자동으로 객체간 관계를 맺어주도록 해준다. 

개발할때 편리하다. 

## UserDetail 관계 매핑 

이제 UserDetail 에 대한 매핑을 다음과 같이 만들어준다. 

```java
package com.schooldevops.practical.simpleboard.entity;

import com.schooldevops.practical.simpleboard.constants.Role;
import com.schooldevops.practical.simpleboard.dto.UserDetailDto;
import com.schooldevops.practical.simpleboard.entity.converter.LocalDateTimeConverter;
import lombok.*;

import javax.persistence.*;
import java.time.LocalDateTime;

@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
@Table(name = "UserDetail")
public class UserDetail {

    @Id
    @Column(name = "id", unique = true)
    private String id;

    @Column
    private String nick;

    @Column
    private String avatarImg;

    @Column
    private String category;

    @Enumerated(EnumType.STRING)
    @Column
    private Role role;

    @Column
    @Convert(converter = LocalDateTimeConverter.class)
    private LocalDateTime joinedAt;

    @Column
    @Convert(converter = LocalDateTimeConverter.class)
    private LocalDateTime modifiedAt;

    @OneToOne
    @MapsId
    private User user;

    @Transient
    public UserDetailDto getDTO() {
        return UserDetailDto.builder()
                .id(this.id)
                .nick(this.nick)
                .avatarImg(this.avatarImg)
                .category(this.category)
                .role(this.role)
                .joinedAt(this.joinedAt)
                .modifiedAt(this.modifiedAt)
                .build();

    }
}

```

내용은 길지만 중요한 부분은 @OneToOne 과 @MapsId 부분이다. 

- @OneToOne: UserDetail 과 User 관계는 1:1임을 선언한다. 
- @MapId: 이것은 UserDetail 이 User와 연관을 맺을때 User의 기본키와 매핑됨을 의미한다. 1:1 관계는 일반적으로 기본키를 동일하게 저장한다. 이를 기본키 공유라고도 부른다. 

### 생성된 테이블 살펴보기. 

```java
Hibernate: 
    
    create table user (
       id varchar(255) not null,
        birth varchar(255),
        created_at datetime(6),
        name varchar(255),
        user_detail_user_id varchar(255),
        primary key (id)
    ) engine=InnoDB
```
user 테이블이 생성되었다. 

```java
Hibernate: 
    
    create table user_detail (
       avatar_img varchar(255),
        category varchar(255),
        joined_at datetime(6),
        modified_at datetime(6),
        nick varchar(255),
        role varchar(255),
        user_id varchar(255) not null,
        primary key (user_id)
    ) engine=InnoDB
```
이제 userDetail 테이블이 생성이 되었다. 

우선 이상한 점은 user_id 라고 해서 기본키가 설정이 되었다. user 테이블은 id 이고, user_detail 테이블은 user_id 가 기본키로 생성되었다. 

```java
Hibernate: 
    
    alter table user 
       add constraint FK64xvo1yt6454tgj3ysjtuym4e 
       foreign key (user_detail_user_id) 
       references user_detail (user_id)
```

이제는 Foreign key 생성부분이다. 

user 테이블에 foreign key 가 생성이 되었고, user_detail 의 user_id 를 참조하도록 생성이 되었다. 

```java
Hibernate: 
    
    alter table user_detail 
       add constraint FKc2fr118twu8aratnm1qop1mn9 
       foreign key (user_id) 
       references user (id)
```

마지막으로 user_detail 테이블에 foreign key 가 생성되었고, 참조는 user 테이블에 id를 참조하도록 생성이 되었따. 

### 비고하기. 

우리가 원하는 그림이 이것이 맞는 것인지 생각해보자. 

사실 1:1 매핑이지만 우리가 원하는 그림은 위와같이 테이블과 foreign key 가 생성되는 것이 아니다. 

우리에게 필요한 것은 다음과 같다. 

- user 테이블
- user_detail 테이블 
- user_detail 테이블의 기본키는 id 로 생성
- user_detail 테이블에 기본키를 생성하고, 이는 user테이블의 id 를 참조하도록 생성

## 매핑 변경하기. 

### mappedBy 이용하기. 

이제 mappedBy 를 이용할 차례이다. 좀전에 테이블 생성시에 양쪽에 foreign key 가 생성되었음을 알 수 있다. 

테이블 릴레이션에는 주, 종 관계가 존재한다. 우리가 생성하고자 하는 테이블은 다음과 같은 주 종 관계가 있다. 

- 주: user테이블, user 데이터가 존재해야 user_detail 데이터도 존재가 가능하다. 
- 종: user_detail테이블, user가 없다면 user_detail 도 없으므로 user_detail 테이블에 foreign_key 로 user 테이블의 id 를 참조하도록 관계가 설정이 된다. 

위와 같이 DBMS 입장에서는 종에 해당하는 테이블이 주의 primary key 를 참조키로 저장하게 된다. 

예) 
- 학생 - 수 이 있다고 생각하자. 학생 1명이 여러 수강을 하게 된다. 
- 여기서 주는 학생이다. 종은 수이다. 수강은 학생 id 를 가지게 된다. 즉 연관관계를 소유한 쪽은 수강쪽이다. 

위 예와 같이 이러한 관계가 무엇이고, 이 관계의 정보를 가진 쪽이 어디인지를 알려주는 것이 mappedBy 속성이다. 

참고: 일반적으로 구글링 해보면 연관관계 주인이라는 말이 나오는데, 틀린말은 아니지만 매우 헛갈리는 용어라고 생각한다. 좀더 이해가 쉽게 하기 위해서는 연관관계 소유자 혹은 Foreign Key 를 소유자 라고 하는 것이 어떨지 싶다. 

mappedBy 는 User 측에 설정한다. 이 의미는 User 은 주에 해당하는 엔터티 이고, 이 주와 관계를 저장하고 있는 측은 UserDetail 엔터티임을 JPA 에게 알려준다. 

#### User 매핑 변경하기. 

그럼 User.java 내용을 다음과 같이 수정해주자. 

```java
    @OneToOne(mappedBy = "user", fetch = FetchType.LAZY, cascade = CascadeType.ALL, orphanRemoval = true)
    private UserDetail userDetail;
```

추가된 것은 mappedBy="user" 이다. 

이 의미는 UserDetail 에 user 라는 속성에 연관관계가 매핑되어 있다는 의미로 해석하면 될것같다. 

#### 생성 결과 확인하기. 

```java
Hibernate: 
    
    create table user (
       id varchar(255) not null,
        birth varchar(255),
        created_at datetime(6),
        name varchar(255),
        primary key (id)
    ) engine=InnoDB

Hibernate: 
    
    create table user_detail (
       avatar_img varchar(255),
        category varchar(255),
        joined_at datetime(6),
        modified_at datetime(6),
        nick varchar(255),
        role varchar(255),
        user_id varchar(255) not null,
        primary key (user_id)
    ) engine=InnoDB

Hibernate: 
    
    alter table user_detail 
       add constraint FKc2fr118twu8aratnm1qop1mn9 
       foreign key (user_id) 
       references user (id)
```

위와 같이 테이블과 user_detail에 Foreign Key 가 생성이 되었다.

우리가 원했던 그림을 다시한번 보자. 

- user 테이블 (OK)
- user_detail 테이블 (OK) 
- user_detail 테이블의 기본키는 id 로 생성 (X)
- user_detail 테이블에 기본키를 생성하고, 이는 user테이블의 id 를 참조하도록 생성 (OK)

## user_detail 테이블의 기본키를 id 로 생성해보자. 

UserDetail.java 내용을 다음과 같이 수정하자. 

```java
    @JoinColumn(name = "id", foreignKey = @ForeignKey(name = "FK_UserDetail_to_user"))
    @OneToOne
    @MapsId
    private User user;
```

추가한 부분은 @JoinColumn 이다. 

- @JoinColumn: foreign key 칼럼의 이름을 지정해준다. 
- foreignKey = ... : foreign key 는 위 결과처럼 FKc2fr118twu8aratnm1qop1mn9 이름이 이렇게 생성이 된다. 어것도 나쁘지 않지만 명시적으로 지정해 줄 수 있다. 우리 예제에서는 FK_UserDetail_to_user 로 잡아보자. 

### 결과 확인하기 

```java
Hibernate: 
    
    create table user (
       id varchar(255) not null,
        birth varchar(255),
        created_at datetime(6),
        name varchar(255),
        primary key (id)
    ) engine=InnoDB

Hibernate: 
    
    create table user_detail (
       id varchar(255) not null,
        avatar_img varchar(255),
        category varchar(255),
        joined_at datetime(6),
        modified_at datetime(6),
        nick varchar(255),
        role varchar(255),
        primary key (id)
    ) engine=InnoDB

Hibernate: 
    
    alter table user_detail 
       add constraint FK_UserDetail_to_user 
       foreign key (id) 
       references user (id)
``` 

우리가 원하는 대로 생성이 되었다. 

mappedBy의 사용법을 확인했고, 그리고 JoinColumn 으로 테이블을 생성해 보았다.

JPA 의 매핑역시 우리의 상식에 벗어나지 않는다. 일반적인 테이블 생성 원리를 그대로 JPA 매핑도 가져가고 있음을 알 수 있다. 
 

