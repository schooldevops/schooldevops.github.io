---
layout: post
title:  "JPA기초 ManyToMany with Entity"
date:   2021-02-25 17:11:49 +0900
categories: Java
tags: [Java, JPA, spring, springboot, database, mysql]
toc: true
---

# ManyToMany

JPA 가 직접 만들어주는 엔터티와 테이블 말고, 우리가 직접 엔터티를 만들고, 테이블을 생성하고자 할 때가 있다. 

## 시나리오. 

- 사용자 정보를 저장하는 User가 있다.  
- 사용자는 다양한 클럽에 가입할 수 있다.  
- 클럽을 저장하는 Club가 있다.  
- Club도 여러 사용자를 가질 수 있다. 
- 클럽에 포함된 사용자를 나타내는 ClubUsers가 있다. 
- ClubUsers 는 유저의 점수, 상태를 저장할 수 있다.  

이러한 시나리오가 전형적인 M:N 관계이다. 이러한 관계는 연관 테이블을 생성하여 비즈니스 니즈를 해결하는 것이 일반적이다.

또한 M:N 이면서 해당 ClubUsers 만의 별도의 속성을 가지고자 할 때 우리가 생성한 ClubUser를 직접 컨트롤 할 수 있다.  

<img width="514" alt="ManyToManyV201" src="https://user-images.githubusercontent.com/66154381/109124310-1f5a3900-778e-11eb-816f-18a87b002f86.png">

위 다이어그램에서 보는 바와 같이 club_id, user_id 를 각각 기본키로 하는 club_users 이라는 테이블으 생성 되었고. 

user : club_users = 1 : N 관계가 형성이 된다. 

group : club_users = 1 : N 관계가 형상이 된다. 

club_users 에는 클럽 유저의 레벨, 상태, 스코어, 가입일, 수정일 등을 가질 수 있다. 

이렇게 중간 테이블을 만들어서 연관을 맺어주면 쉽게 M:N 관계를 설정할 수 있다. 

## Glub 엔터티 생성하기. 

지금까지와 같이 Club 엔터티를 생성하자. 

```java
package com.schooldevops.practical.simpleboard.entity;

import com.schooldevops.practical.simpleboard.constants.ClubLevel;
import com.schooldevops.practical.simpleboard.dto.ClubDto;
import com.schooldevops.practical.simpleboard.entity.converter.LocalDateTimeConverter;
import lombok.*;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.Set;
import java.util.stream.Collectors;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Entity
@Table(name = "club")
public class Club {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", unique = true)
    private Long id;

    @Column
    private String name;

    @Column
    @Enumerated(value = EnumType.STRING)
    private ClubLevel clubLevel;

    @Column
    @Convert(converter = LocalDateTimeConverter.class)
    private LocalDateTime createdAt;

    @Transient
    public ClubDto getDTO() {
        return ClubDto.builder()
                .id(this.id)
                .name(this.name)
                .clubLevel(this.clubLevel)
                .createdAt(this.createdAt)
                .build();
    }
}

```

위와 같이 만들어 주었다. 

엔터티를 생성하는 방법은 동일하다.

## ClubUsers 엔터티 생성하기. 

이제는 ClubUsers 엔터티를 생성할 것이다. 

이때 ClubUsers 는 기본키로 2개의 키를 가지고 있다. 그러므로 2개 이상의 기본키를 사용하기 위해서는 키를 나타내는 클래스를 별도로 지정해 주어야한다. 

### ClubUsersKey 생성하기. 

```java
package com.schooldevops.practical.simpleboard.entity;

import lombok.*;

import javax.persistence.Column;
import javax.persistence.Embeddable;
import java.io.Serializable;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EqualsAndHashCode
@Embeddable
public class ClubUsersKey implements Serializable {

    @Column(name = "club_id")
    private Long clubId;

    @Column(name = "user_id")
    private String userId;
}

``` 

- @Embeddable: 이를 통해서 엔터티에 임베디드 되도록 설정한다. 
- Serializable: 키는 반드시 Serializable 를 상속해야한다. 키 값을 서로 비교하기 위해서는 직렬화된 결과가 필요하다. 
- @EqualsAndHashCode: 이 어노테이션으로 키가 동일함을 비교하는 기준을 만들 수 있다. 
- @Column(name = "club_id"): 생성할 기본 키의 필드 이름을 지정한다. 
- @Column(name = "user_id"): 생성할 기본 키의 필드 이름을 지정한다. 

이렇게 키를 생성했다. 

### ClubUsers 생성하기. 

```java
package com.schooldevops.practical.simpleboard.entity;

import com.schooldevops.practical.simpleboard.constants.ClubUserLevel;
import com.schooldevops.practical.simpleboard.constants.ClubUserStatus;
import com.schooldevops.practical.simpleboard.dto.ClubUsersDto;
import com.schooldevops.practical.simpleboard.entity.converter.LocalDateTimeConverter;
import lombok.*;

import javax.persistence.*;
import java.time.LocalDateTime;

@AllArgsConstructor
@NoArgsConstructor
@Builder
@Setter
@Getter
@Entity
public class ClubUsers {

    @Id
    private ClubUsersKey id;

    @Enumerated(value = EnumType.STRING)
    @Column(nullable = false)
    private ClubUserLevel clubUserLevel;

    @Column(nullable = false)
    private Integer score;

    @Enumerated(value = EnumType.STRING)
    @Column(nullable = false)
    private ClubUserStatus clubUserStatus;

    @Convert(converter = LocalDateTimeConverter.class)
    @Column(nullable = false)
    private LocalDateTime createdAt;

    @Convert(converter = LocalDateTimeConverter.class)
    private LocalDateTime modifiedAt;

    public ClubUsersDto getDTO() {
        return ClubUsersDto.builder()
                .id(this.id)
                .clubUserLevel(this.clubUserLevel)
                .score(this.score)
                .clubUserStatus(this.clubUserStatus)
                .createdAt(this.createdAt)
                .modifiedAt(this.modifiedAt)
                .build();
    }
}

```

ClubUser 를 위한 기본적인 설정을 해보았다. @Embeddable 로 정의한 ClusterKey 를 ID로 지정한것 이외에는
지금까지 작성한 방식과 동일하다. 

이렇게 해서 생성하면 ClubUsers 는 관계가 없는 일반 테이블로 생성된다. 

## 연관 설정하기. 

이제 연관을 하나씩 설정해보자. 

user - club_user 은 서로 주-종 관계이며 user가 있어야 club_user도 존재할 수 있다. 

그러므로 club_user 에 ForeiginKey 가 설정이 되며, user의 아이디를 참조하도록 설정해야한다. 

즉 연관관계를 소유한 곳은 ClubUsers 엔터티가 될 것이다. 

club - club_user 역시 user와 동일하게 주-종 관계이며 club가 있어야 club_user도 존재할 수 있다. 

그러므로 club_user에 ForeignKey 가 설정이 되며, club의 아이디를 참조하도록 설정해야한다. 

즉, 연관관계를 소유한 곳은 ClubUsers 엔터티가 될 것이다. 

위 내용을 바탕으로 연관을 설정하면 된다. 

### ClubUsers 엔터티에 연관맺기. 

ClubUsers.java 파일에 다음과 같이 코드를 추가한다. 

```java
... 중간 생략 ...
    @ManyToOne
    @MapsId("clubId")
    @JoinColumn(name = "club_id")
    private Club club;

    @ManyToOne
    @MapsId("userId")
    @JoinColumn(name = "user_id")
    private User user;
... 중간 생략 ...
``` 

- @ManyToOne: 위처럼 club:club_user = 1:N 이므로 @ManyToOne 로 설정했다. 
- @MapsId("clubId"): 이것은 club의 id 값으로 club_user의 테이블의 기본키로 하겠다는 의미이다. 
- @JoinColumn(name = "club_id"): 조인될 칼럼 이름을 지정한다. club_id 로 Foreign Key 가 설정된다. 

### Club 엔터티 연관 맺기.

이제 Club 엔터티에 연관을 생성해보자. 

```java
... 중간 생략 ...
    @OneToMany(mappedBy = "club")
    private Set<ClubUsers> clubUsers;
... 중간 생략 ...
```

- @OneToMany: club:club_user = 1:N 이므로 OneToMany 로 설정했다. 
- mappedBy = "club": 위 설정은 연관관계를 소유한곳, 즉 Foreign Key가 만들어져야 할 곳이 club_user 임을 의미한다. 
- Set: 클럽 유저의 경우 순서와 관계 없으므로 Set 으로 지정했다. 키 비교시 성능이 향상될 것이다. 

### User 엔터티 연관 맺기. 

이제 User 도 연관을 생성해보자. 

```java

... 중간 생략 ...
    @OneToMany(mappedBy = "user")
    private Set<ClubUsers> clubUsers;
... 중간 생략 ...
```
- @OneToMany: user:club_user = 1:N 이므로 OneToMany 로 설정했다. 
- mappedBy = "user": 위 설정은 연관관계를 소유한곳, 즉 Foreign Key가 만들어져야 할 곳이 club_user 임을 의미한다. 
- Set: 클럽 유저의 경우 순서와 관계 없으므로 Set 으로 지정했다. 키 비교시 성능이 향상될 것이다. 

## 결과 확인하기. 

지금까지 만들어진 테이블을 확인해보자. 

### club 테이블 

```java
Hibernate: 
    
    create table club (
       id bigint not null auto_increment,
        club_level varchar(255),
        created_at datetime(6),
        name varchar(255),
        primary key (id)
    ) engine=InnoDB
```

### club_users 테이블 

```java
Hibernate: 
    
    create table club_users (
       club_id bigint not null,
        user_id varchar(255) not null,
        club_user_level varchar(255) not null,
        club_user_status varchar(255) not null,
        created_at datetime(6) not null,
        modified_at datetime(6),
        score integer not null,
        primary key (club_id, user_id)
    ) engine=InnoDB
```

기본키로 club_id, user_id 가 생성되었다. 

### Foreign Key

```java
Hibernate: 
    
    alter table club_users 
       add constraint FKgx7pn8lywnipyc2xgs07o6h75 
       foreign key (club_id) 
       references club (id)

Hibernate: 
    
    alter table club_users 
       add constraint FKnjj4jo11tg1cw930dw3xh3ea1 
       foreign key (user_id) 
       references user (id)
```

위와 같이 foreign key 가 club_users 에 생성이 되었다. 

하나는 club의 id와 연관이 생성이 되었고, 다른 하나는 user의 id와 연관이 생성 되었다. 

이렇게 해서 M:N 관계를 JPA 에 적용해 보았다. 

연관된 엔터티가 특정 칼럼을 소유해야할 때에는 ClubUsers 와 같이 엔터티를 생성하고, 이를 직접 활용하는 방법을 이용하면 좋다. 
