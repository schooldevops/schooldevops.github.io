---
layout: post
title:  "JPA기초 Entity Event Hooking"
date:   2021-02-25 17:12:49 +0900
categories: Java
tags: [Java, JPA, spring, springboot, database, mysql, event hooking]
toc: true
---

# Entity Event Hooking

JPA 를 사용한다는 것은 DB CRUD 를 JPA 를 통해 수행한다는 의미이다. 

그러므로 JPA 에서는 다양한 이벤트에 대해서 CallBack 을 받을 수 있도록 하는 여러 방법을 제공하고 있다. 

이러한 이벤트는 CRUD 가 일어나기 전, 후에 각각 이벤트를 통지 받을 수 있으며 여러가지 방법을 제공하고 있다. 

## Entity Event 

- @PrePersist: 새로운 엔터티가 저장되기 전에 호출된다. 
- @PostPersist: 새로운 엔터티가 저장되고 난 후에 호출 된다.
- @PreRemove: 엔터티가 삭제 되기 전에 호출된다. 
- @PostRemove: 엔터티가 삭제 되고 난 후에 호출된다. 
- @PreUpdate: 엔터티가 업데이트 되기 전에 호출된다. 
- @PostUpdate: 엔터티가 업데이트 되고난 후에 호출된다. 
- @PostLoad: 엔터티가 로드된 후에 호출된다. 

위 엔터티에 대한 이벤트를 확인해 보면, 저장전/후, 삭제전/후, 업데이트전/후, 조회후 이렇게 호출된다는 의미이다. 

각 상황에 따라서 이벤트의 속성값을 조회하면, 엔터티가 DB에 오퍼레이션이 발생하는 순간 엔터티의 값들을 조회하고, 필요한 작업을 수행할 수 있다. 

보통 이러한 작업을 후킹이라고 부른다. 

## Entity에 Event 걸어주기. 

User.java 에 다음과 같은 코드를 추가해보자. 

```java
    @PrePersist
    public void callBackPrePersist() {
        log.info("fireEvent PrePersist: ");
    }

    @PostPersist
    public void callBackPostPersist() {
        log.info("fireEvent PostPersist: ");
    }

    @PreRemove
    public void callBackPreRemove() {
        log.info("fireEvent PreRemove");
    }

    @PostRemove
    public void callBackPostRemove() {
        log.info("fireEvent PostRemove");
    }

    @PreUpdate
    public void callBackPreUpdate() {
        log.info("fireEvent PreUpdate");
    }

    @PostUpdate
    public void callBackPostUpdate() {
        log.info("fireEvent PostUpdate");
    }

    @PostLoad
    public void callBackPostLoad() {
        log.info("fireEvent PostLoad");
    }
```

위 상황과 같이 User 엔터티 객체를 DB 에 저장, 수정, 조회, 삭제를 수행할때 어떠한 동작이 수행되는지 알 수 있다. 

```java
2020-11-10 22:30:41.725  INFO 3851 --- [nio-8080-exec-7] c.s.practical.simpleboard.entity.User    : fireEvent PrePersist: 
Hibernate: 
    select
        userdetail0_.id as id1_7_0_,
        userdetail0_.avatar_img as avatar_i2_7_0_,
        userdetail0_.category as category3_7_0_,
        userdetail0_.joined_at as joined_a4_7_0_,
        userdetail0_.modified_at as modified5_7_0_,
        userdetail0_.nick as nick6_7_0_ 
    from
        user_detail userdetail0_ 
    where
        userdetail0_.id=?
2020-11-10 22:30:41.726 TRACE 3851 --- [nio-8080-exec-7] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [kido]
Hibernate: 
    insert 
    into
        user
        (birth, created_at, name, id) 
    values
        (?, ?, ?, ?)
2020-11-10 22:30:41.729 TRACE 3851 --- [nio-8080-exec-7] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [770605]
2020-11-10 22:30:41.729 DEBUG 3851 --- [nio-8080-exec-7] tributeConverterSqlTypeDescriptorAdapter : Converted value on binding : 2020-11-10T22:30:41.718 -> 2020-11-10 22:30:41.718
2020-11-10 22:30:41.729 TRACE 3851 --- [nio-8080-exec-7] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [TIMESTAMP] - [2020-11-10 22:30:41.718]
2020-11-10 22:30:41.729 TRACE 3851 --- [nio-8080-exec-7] o.h.type.descriptor.sql.BasicBinder      : binding parameter [3] as [VARCHAR] - [kido]
2020-11-10 22:30:41.729 TRACE 3851 --- [nio-8080-exec-7] o.h.type.descriptor.sql.BasicBinder      : binding parameter [4] as [VARCHAR] - [kido]
2020-11-10 22:30:41.732  INFO 3851 --- [nio-8080-exec-7] c.s.practical.simpleboard.entity.User    : fireEvent PostPersist: 
Hibernate: 
    insert 
    into
        user_detail
        (avatar_img, category, joined_at, modified_at, nick, id) 
    values
        (?, ?, ?, ?, ?, ?)
```

로그 결과를 확인해보면 다음과 같이 처리된다. 

1. fireEvent PrePersist
2. select / insert 수행
3. fireEvent PostPersist

위 순서대로 수행이 됨을 확인할 수 있다. 

지금까지 수행한 작업은 Entity 객체에 해당 어노테이션을 걸어 주었으며, 데이터를 삽입시 수행됨을 확인할 수 있다. 

### 수정 요청 처리하기. 

```java
2020-11-10 22:34:22.490  INFO 3851 --- [nio-8080-exec-1] c.s.practical.simpleboard.entity.User    : fireEvent PreUpdate
Hibernate: 
    update
        user 
    set
        birth=?,
        created_at=?,
        name=? 
    where
        id=?
2020-11-10 22:34:22.491 TRACE 3851 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [770707]
2020-11-10 22:34:22.491 DEBUG 3851 --- [nio-8080-exec-1] tributeConverterSqlTypeDescriptorAdapter : Converted value on binding : 2020-11-10T22:30:41.718 -> 2020-11-10 22:30:41.718
2020-11-10 22:34:22.491 TRACE 3851 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [TIMESTAMP] - [2020-11-10 22:30:41.718]
2020-11-10 22:34:22.491 TRACE 3851 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicBinder      : binding parameter [3] as [VARCHAR] - [kido2]
2020-11-10 22:34:22.491 TRACE 3851 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicBinder      : binding parameter [4] as [VARCHAR] - [kido]
2020-11-10 22:34:22.494  INFO 3851 --- [nio-8080-exec-1] c.s.practical.simpleboard.entity.User    : fireEvent PostUpdate
```

로그 결과를 확인하면 다음과 같다. 

1. fireEvent PreUpdate 이벤트 호출
2. Update 수행
3. fireEvent PostUpdate 이벤트 호출

위 순서처럼 업데이트 수정전/후 로 호출이 일어남을 확인할 수 있다. 

### 조회 요청 처리하기. 

조회를 수행하면 어떠한 이벤트가 발생하는지 확인해보자. 

```java
Hibernate: 
    select
        userdetail0_.id as id1_7_0_,
        userdetail0_.avatar_img as avatar_i2_7_0_,
        userdetail0_.category as category3_7_0_,
        userdetail0_.joined_at as joined_a4_7_0_,
        userdetail0_.modified_at as modified5_7_0_,
        userdetail0_.nick as nick6_7_0_ 
    from
        user_detail userdetail0_ 
    where
        userdetail0_.id=?
2020-11-10 22:45:08.690 TRACE 3851 --- [nio-8080-exec-3] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [kido]
2020-11-10 22:45:08.692 TRACE 3851 --- [nio-8080-exec-3] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([avatar_i2_7_0_] : [VARCHAR]) - [http://kido.com/img.png]
2020-11-10 22:45:08.692 TRACE 3851 --- [nio-8080-exec-3] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([category3_7_0_] : [VARCHAR]) - [Computer Science]
2020-11-10 22:45:08.692 TRACE 3851 --- [nio-8080-exec-3] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([joined_a4_7_0_] : [TIMESTAMP]) - [2020-11-10 22:30:41.718]
2020-11-10 22:45:08.692 DEBUG 3851 --- [nio-8080-exec-3] tributeConverterSqlTypeDescriptorAdapter : Converted value on extraction: 2020-11-10 22:30:41.718 -> 2020-11-10T22:30:41.718
2020-11-10 22:45:08.693 TRACE 3851 --- [nio-8080-exec-3] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([modified5_7_0_] : [TIMESTAMP]) - [2020-11-10 22:30:41.718]
2020-11-10 22:45:08.693 DEBUG 3851 --- [nio-8080-exec-3] tributeConverterSqlTypeDescriptorAdapter : Converted value on extraction: 2020-11-10 22:30:41.718 -> 2020-11-10T22:30:41.718
2020-11-10 22:45:08.693 TRACE 3851 --- [nio-8080-exec-3] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([nick6_7_0_] : [VARCHAR]) - [ddo]
2020-11-10 22:45:08.693  INFO 3851 --- [nio-8080-exec-3] c.s.practical.simpleboard.entity.User    : fireEvent PostLoad
```

1. select 가 수행된다. 
2. fireEvent PostLoad 가 호출된다. 

조회의 경우 PostLoad 가 호출이 일어난다. 

### 삭제 요청 처리하기. 

이번에는 삭제인경우를 확인해보자.

```java
2020-11-10 22:46:43.699  INFO 3851 --- [nio-8080-exec-6] c.s.practical.simpleboard.entity.User    : fireEvent PreRemove
Hibernate: 
    delete 
    from
        user_role 
    where
        user_id=?
2020-11-10 22:46:43.700 TRACE 3851 --- [nio-8080-exec-6] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [kido]
Hibernate: 
    delete 
    from
        user_detail 
    where
        id=?
2020-11-10 22:46:43.703 TRACE 3851 --- [nio-8080-exec-6] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [kido]
Hibernate: 
    delete 
    from
        user 
    where
        id=?
2020-11-10 22:46:43.707 TRACE 3851 --- [nio-8080-exec-6] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [kido]
2020-11-10 22:46:43.711  INFO 3851 --- [nio-8080-exec-6] c.s.practical.simpleboard.entity.User    : fireEvent PostRemove

```

1. fireEvent PreRemove 가 호출된다. 
2. 삭제 및 cascade 된 객체가 모두 삭제된다. 
3. fireEvent PostRemove 가 호출된다.

보는바와 같이 삭제도 삭제 전/후가 호출됨을 확인 할 수 있다. 

## Event Listener 생성하기. 

이번에는 Entity에 직접 거는 대신에 EntityEventListener 를 생성하는 방법을 알아보자. 

새로운 클래스인 RoleEventTrailListener.java 를 생성하자. 

```java
package com.schooldevops.practical.simpleboard.entity.listener;

import com.schooldevops.practical.simpleboard.entity.RoleEntity;
import lombok.extern.slf4j.Slf4j;

import javax.persistence.*;

@Slf4j
public class RoleEventTrailListener {

    @PrePersist
    @PreUpdate
    @PreRemove
    private void beforeAnyUpdate(RoleEntity role) {
        if (role.getId() == null) {
            log.info("Brand New Role Insert Event. ");
        } else {
            log.info("Update or delete Role: " + role.getId());
        }
    }

    @PostPersist
    @PostUpdate
    @PostRemove
    private void afterAnyUpdate(RoleEntity role) {
        log.info("after add/update/delete complete for role: " + role.getId());
    }

    @PostLoad
    private void afterLoad(RoleEntity role) {
        log.info("role loaded from database: " + role.getId());
    }
}

```

우리는 3개의 메소드에 각각 이벤트 어노테이션을 걸어주었다. 

```java
    @PrePersist
    @PreUpdate
    @PreRemove
    private void beforeAnyUpdate(RoleEntity role)
```

이것처럼 beforeAndUpdate 메소드에는 저장전, 수정전, 삭제전 처리를 할 수 있도록 콜백 메소드를 만들었다.

```java
    @PostPersist
    @PostUpdate
    @PostRemove
    private void afterAnyUpdate(RoleEntity role)
```

이제는 afterAnyUpdate 메소드에는 저장후, 수정후, 삭제후 처리를 할 수 있도록 콜백 메소드를 만들었다. 

```java
    @PostLoad
    private void afterLoad(RoleEntity role)
```

afterLoad 는 엔터티를 조회한 후 호출이 된다. 

위와 같이 이벤트를 만들 수 있으며, 우리는 Role 엔터티가 이벤트를 받을 수 있도록 할 것이기 때문에 

RoleEntity.java 에 다음과 같이 어노테이션을 붙여준다. 

```java
@EntityListeners(RoleEventTrailListener.class)
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Getter
... 생략 
```

@EntityListeners(RoleEventTrailListener.class) 을 걸어주면 RoleEntity 의 CRUD 상황에 따라 이벤트가 발생함을 확인할 수 있다. 

### 생성 이벤트. 

```java
2020-11-10 22:53:23.133  INFO 3851 --- [nio-8080-exec-8] c.s.p.s.e.l.RoleEventTrailListener       : Brand New Role Insert Event. 
Hibernate: 
    insert 
    into
        role
        (name) 
    values
        (?)
2020-11-10 22:53:23.134 TRACE 3851 --- [nio-8080-exec-8] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [Administrator]
2020-11-10 22:53:23.136  INFO 3851 --- [nio-8080-exec-8] c.s.p.s.e.l.RoleEventTrailListener       : after add/update/delete complete for role: 2
```

1. Brand New Role Insert Event. 가 호출됨
2. insert 가 수행됨
3. after add/update/delete complete for role: 2 가 호출됨. 

위와 같이 이벤트가 호출 되었다. 

우리가 지정한 @PrePersist 와 @PostPersist 에 대해서 이벤트가 발생함을 확인할 수 있다. 

### 조회 이벤트 확인하기. 

```java
Hibernate: 
    select
        roleentity0_.id as id1_3_0_,
        roleentity0_.name as name2_3_0_ 
    from
        role roleentity0_ 
    where
        roleentity0_.id=?
2020-11-10 22:55:42.410 TRACE 3851 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [2]
2020-11-10 22:55:42.413 TRACE 3851 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([name2_3_0_] : [VARCHAR]) - [Administrator]
2020-11-10 22:55:42.413 TRACE 3851 --- [nio-8080-exec-1] org.hibernate.type.CollectionType        : Created collection wrapper: [com.schooldevops.practical.simpleboard.entity.RoleEntity.users#2]
2020-11-10 22:55:42.413  INFO 3851 --- [nio-8080-exec-1] c.s.p.s.e.l.RoleEventTrailListener       : role loaded from database: 2
```

1. select 수행
2. role loaded from database: 2 가 호출됨. 

select 인경우 @PostLoaded 가 호출됨을 확인했다. 

## Wrapup

이처럼 JPA 엔터티의 라이프사이클에 따라 다양한 이벤트 훅을 활용해보았고, 엔터티에 직접 연동하는 방법, EventListener 를 이용하는 방법을 각각 알아 보았다. 

이벤트 훅은 데이터의 변경 사항을 기록하거나, 중간에 처리해야할 사항들을 적절히 활용할 수 있도록 해준다. 
  