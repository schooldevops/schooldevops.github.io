---
layout: post
title:  "JPA기초 Derived Query"
date:   2021-02-25 17:13:49 +0900
categories: Java
tags: [Java, JPA, spring, springboot, database, mysql, query]
toc: true
---

# Derived Query

스프링 JPA 는 메소드 이름으로 쿼리를 생성하는 기능을 제공한다. 이를 파생된 쿼리라고 하며 엔터티 속성들을 다양하게 활용하여 쿼리를 생성해 낼 수 있다. 

이번 아티클에서는 이름을 통한 파생쿼리를 이용하여 우리가 원하는 쿼리를 어떻게 작성할 수 있는지 살펴 볼 것이다. 

## Query Sample 살펴보기. 

아래는 JPA 가이드 샘플 내용이다. [Sample](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference)

### Distinct

findDistinctByLastnameAndFirstname

lastname, firstname과 동일한 결과를 찾아서 중복제거(distinct)를 수행한다.  

```java
select distinct …​ where x.lastname = ?1 and x.firstname = ?2
````

### And

findByLastnameAndFirstname

Where 절에 들어갈 criteria 를 and 로 연결한다. 

```java
… where x.lastname = ?1 and x.firstname = ?2
```

### Or

findByLastnameOrFirstname

Where 절에 들어갈 criteria 를 or 로 연결한다.
 
```java
… where x.lastname = ?1 or x.firstname = ?2
```

### Is, Equals

findByFirstname,findByFirstnameIs,findByFirstnameEquals

exact 매칭을 수행한다. 

```java
… where x.firstname = ?1
```

### Between

findByStartDateBetween

between 절을 이용하여 조회한다. 

```java
… where x.startDate between ?1 and ?2
```

### LessThan

findByAgeLessThan

```java
… where x.age < ?1
```

### LessThanEqual

findByAgeLessThanEqual

```java
… where x.age <= ?1
```

### GreaterThan

findByAgeGreaterThan

```java
… where x.age > ?1
```

### GreaterThanEqual

findByAgeGreaterThanEqual

```java
… where x.age >= ?1
```

### After

findByStartDateAfter

```java
… where x.startDate > ?1
```

### Before

findByStartDateBefore

```java
… where x.startDate < ?1
```

### IsNull, Null

findByAge(Is)Null

Age가 NULL 인 데이터를 조회한다.  

```java
… where x.age is null
```

### IsNotNull, NotNull

findByAge(Is)NotNull

Age가 NULL 이 아닌 데이터를 조회한다. 

```java
… where x.age not null
```

### Like

findByFirstnameLike

```java
… where x.firstname like ?1
```

### NotLike

findByFirstnameNotLike

```java
… where x.firstname not like ?1
```

### StartingWith

findByFirstnameStartingWith

```java
… where x.firstname like ?1 (parameter bound with appended %)
```

### EndingWith

findByFirstnameEndingWith

```java
… where x.firstname like ?1 (parameter bound with prepended %)
```

### Containing

findByFirstnameContaining

```java
… where x.firstname like ?1 (parameter bound wrapped in %)
```

### OrderBy

findByAgeOrderByLastnameDesc

```java
… where x.age = ?1 order by x.lastname desc
```

### Not

findByLastnameNot

```java
… where x.lastname <> ?1
```

### In

findByAgeIn(Collection<Age> ages)

```java
… where x.age in ?1
```

### NotIn

findByAgeNotIn(Collection<Age> ages)

```java
… where x.age not in ?1
```

### True

findByActiveTrue()

```java
… where x.active = true
```

### False

findByActiveFalse()

```java
… where x.active = false
```

### IgnoreCase

findByFirstnameIgnoreCase

```java
… where UPPER(x.firstname) = UPPER(?1)
```

위 내용을 바탕으로 지금까지 우리가 만들어둔 데이터를 조회해 보자. 

## Query 수행하기.

### 회원 이름이 'kido' 이고 데이터 생성일이 10일전부터 오늘 사이의 회원을 조회하기. 

List<User> findAllByNameAndCreatedAtBetween(String name, LocalDateTime startDate, LocalDateTime endDate);

- findAll: 모두 조회하라. 
- By: 다음부터 나오는 조건을 비교하라. 
- ByName: 이름이 매칭되는 즉, name = ? 으로 해석된다. 
- And: And 조건
- CreatedAtBetween: createdAt 에 대해서 between 조건을 검사하라.  

위 생성 쿼리 결과는 다음과 같다. 

```java
Hibernate: 
    select
        user0_.id as id1_5_,
        user0_.birth as birth2_5_,
        user0_.created_at as created_3_5_,
        user0_.name as name4_5_ 
    from
        user user0_ 
    where
        user0_.name=? 
        and (
            user0_.created_at between ? and ?
        )
```

우리가 의도한 대로 조회가 되는 것을 확인할 수 있다. 

## Top, First 쿼리

특정 조회 결과에서 1명만 조회하라 혹은 N명 조회하라는 요구사항이 일어날 수 있다. 이때는 Top 혹은 First 를 활용하면 쉽게 해결이 된다. 

List<User> findTop3ByName(String name);

- Top{N}: Top 만 수행하면 1개의 결과만 반환한다. 그러나 N을 지정해주면 지정된 수만큼 가져온다. 
    - First 와도 동일하다. 

```java
Hibernate: 
    select
        user0_.id as id1_5_,
        user0_.birth as birth2_5_,
        user0_.created_at as created_3_5_,
        user0_.name as name4_5_ 
    from
        user user0_ 
    where
        user0_.name=? limit ?
```

쿼리 결과가 Limit 된것을 확인할 수 있다. 

## Sorting

사용자 이름으로 조회를 하는데, 사용자 정보가 생성된 일자를 기준으로 소트한다고 해보자. 

이때 Repository 에는 다음과 같이 작성해준다. 

List<User> findByName(String name, Sort sort);

이름으로 조회하며, 결과를 Sort 하도록 Sort 파라미터를 전달한다. 

### Sort 표현식 생성하기.

Sort 객체를 전달하기 위해서는 아래와 같이 2가지 방법을 이용할 수 있다. 

### TypeSafe 방식으로 생성하기. 

```java
    private Sort getUserSortByCreatedAt(String ascdes) {
        Sort.TypedSort<User> userSortType = Sort.sort(User.class);
        Sort.TypedSort<LocalDateTime> sortBy = userSortType.by(User::getCreatedAt);
        
        if ("DES".equals(ascdes)) {
            return sortBy.descending();
        }
        return sortBy.ascending();
    }
```

이 메소드는 Sort.TypedSort 라는 typeSafe 형태로 생성한 것이다 .

위와 같이 Sort 를 생성하면, code 는 약간 장황하지만, 버그 없는 코드를 만들 수 있다. 

### 필드 이름을 직접 스트링으로 해서 생성. 

```java
    private Sort getUserSortByCreatedAtV2(String ascdes) {
        if ("DES".equals(ascdes)) {
            return Sort.by("createdAt").descending();
        }
        return Sort.by("createdAt").ascending();
    }
``` 

타임 안정성 잇는 코드보다는 간편하지만, ```"``` 안에 있는 필드 이름의 오타로 인해서 오류 유발 가능성이 더 높다. 

무엇을 사용하든지 소트 객체를 만들어서 Repository 에 생성한 메소드 이름에 파라미터로 전달하면 다음과 같은 쿼리 결과를 확인할 수 있다. 

```java
Hibernate: 
    select
        user0_.id as id1_5_,
        user0_.birth as birth2_5_,
        user0_.created_at as created_3_5_,
        user0_.name as name4_5_ 
    from
        user user0_ 
    where
        user0_.name=? 
    order by
        user0_.created_at desc
```

## Paging 

페이징은 비즈니스 요건으로 항상 나타나는 사항중에 하나이다. 

페이징 역시 Sort와 마찬가지로 메소드 파라미터로 전달되며, 결과를 부분적으로 조회할 수 있다. 

```java
Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Pageable pageable);
```

위 메소드는 페이지에 대한 메소드 작성예를 보여준다. 

- Page로 반환하는 경우 Page 요청 내용을 함께 저장할 수 있으므로, 현재 페이지 번호와 한번 조회할때 row 개수등 기본적인 페이징 요건도 함께 저장된다. 
- Slice 는 다음 Slice 가 존재하는지에 대해서 결과를 반환하며 더보기 기능과 같은 쿼리를 작성할때 효과적일 것이다. 
- List 는 단지 페이징 결과를 리스트로 반환하며, 직관적이겠지만 페이징 정보를 담지 않기 때문에 별도로 페이징 요건을 저장해야한다. 

```java
	/**
	 * Returns a {@link Page} of entities meeting the paging restriction provided in the {@code Pageable} object.
	 *
	 * @param pageable
	 * @return a page of entities
	 */
	Page<T> findAll(Pageable pageable);
```

모든 사용자를 조회하고, Page 정책에 따라서 조회하도록 하고 있다. 이 쿼리는 이미 CRUDRepository 에 내장되어 있는 것을 사용할 것이다.  

### Page 요건 생성하기. 

페이지객체를 생성하기 위해서는 다음과 같이 3가지 정적 메소드를 이용할 수 있다. 

```java
	/**
	 * 페이지 요청객체 생성한다. 
	 *
	 * @param page 0 부터 시작하는 페이지 인덱스, 음수가 되면 안된다. 
	 * @param size 0 보다 큰 값으로 페이지당 결과 건수를 지정한다. 
	 * @since 2.0
	 */
	public static PageRequest of(int page, int size) {
		return of(page, size, Sort.unsorted());
	}

	/**
	 * 소트가 적용된 페이지 요청객체 생성 
	 *
	 * @param page 0 부터 시작하는 페이지 인덱스, 음수가 되면 안된다. 
	 * @param size 0 보다 큰 값으로 페이지당 결과 건수를 지정한다. 
	 * @param sort 소트 객체를 전달한다. 널이 되면 안된다. 널이라면 Sort.unsorted() 를 전달하거나 해야한다. 
	 * @since 2.0
	 */
	public static PageRequest of(int page, int size, Sort sort) {
		return new PageRequest(page, size, sort);
	}

	/**
	 * Creates a new {@link PageRequest} with sort direction and properties applied.
	 *
	 * @param page 0 부터 시작하는 페이지 인덱스, 음수가 되면 안된다. 
	 * @param size 0 보다 큰 값으로 페이지당 결과 건수를 지정한다. 
	 * @param sort 소트 객체를 전달한다. 널이 되면 안된다. 널이라면 Sort.unsorted() 를 전달하거나 해야한다. 
	 * @param properties 소트 대상을 넣는다. 널이 되면 안된다. 
	 * @since 2.0
	 */
	public static PageRequest of(int page, int size, Direction direction, String... properties) {
		return of(page, size, Sort.by(direction, properties));
	}
```

아래와 같이 요청 서비스를 만들어보자. 

```java
    public Page<UserDto> findAllUserWithPaging(int page, int sizePerPage) {
        PageRequest pageReq = PageRequest.of(page, sizePerPage);
        Page<User> usersWithPage = userRepository.findAll(pageReq);

        log.info("Total Users: " + usersWithPage.getTotalElements());
        log.info("Total Pages: " + usersWithPage.getTotalPages());

        List<UserDto> users = usersWithPage.getContent().stream().map(user -> user.getDTO()).collect(Collectors.toList());

        return new PageImpl<>(users, usersWithPage.getPageable(), usersWithPage.getTotalElements());
    }
```

- PageRequest.of(page, sizePerPage): 페이지를 위한 표현식을 만든다. 
- findAll(Pageable): 쿼리를 수행한다. 
- userWithPage.getTotlaElements(): 전체 글의 개수를 확인할 수 있다. 
- usersWithPage.getTotalPages(): 총 페이지 개수를 확인할 수 있다. 
- new PageImpl<>(users, usersWithPage.getPageable(), usersWithPage.getTotalElements()) : 요청한 클라이언트로 Page 객체를 전달해야하므로, Page 객체를 생성했다. 

### 결과 보기. 

이제 쿼리와 조회 결과를 한번 살펴 볼 것이다. 

```java
Hibernate: 
    select
        user0_.id as id1_5_,
        user0_.birth as birth2_5_,
        user0_.created_at as created_3_5_,
        user0_.name as name4_5_ 
    from
        user user0_ limit ?,
        ?
```
mysql 로 처리가 가능한 limit ?, ? 로 쿼리가 생성되었다. 이것으로 쿼리 결과가 조회 될 것이다. 

```java
Hibernate: 
    select
        count(user0_.id) as col_0_0_ 
    from
        user user0_
```
totalElement 를 확인할 수 있도록 전체 개수를 확인할 수 있다. 

이 결과로 우리는 토탈 엘리먼트를 확인할 수 있고, 토탈 페이지 개수도 확인할 수 있다. 

```java
{
  "content": [
    {
      "id": "kido3",
      "name": "kido",
      "birth": "770605",
      "createdAt": "2020-11-16T10:15:06.339",
      "userDetail": null,
      "roles": null
    },
    {
      "id": "kido4",
      "name": "kido",
      "birth": "770605",
      "createdAt": "2020-11-16T10:15:09.611",
      "userDetail": null,
      "roles": null
    }
  ],
  "pageable": {
    "sort": {
      "unsorted": true,
      "sorted": false,
      "empty": true
    },
    "pageNumber": 1,
    "pageSize": 2,
    "offset": 2,
    "unpaged": false,
    "paged": true
  },
  "last": false,
  "totalElements": 5,
  "totalPages": 3,
  "sort": {
    "unsorted": true,
    "sorted": false,
    "empty": true
  },
  "first": false,
  "size": 2,
  "number": 1,
  "numberOfElements": 2,
  "empty": false
}
```

보는 바와 같이 위 구조는 다음과 같이 조회된다. 

- content: 페이징 결과 목록을 배열로 저장되어 있다. 
- pageable: 페이지 정보를 저장한다. 
    - sort: 소트 내역을 확인할 수 있다. 
    - pageNumber: 현재 페이지 번호 (우리는 1, 2를 전달했고, 현재는 2페이지의 결과 2개를 가져온 것이다.) 참고: 페이지는 0부터 시작한다. 
    - pageSize: 현재 페이지당 결과 개수이다. 
- last: 마지막 페이지인지 여부 
- totalElements: 총 엘리먼트의 개수 
- totalPage: 총 페이지 개수 
- first: 첫페이지 여부 
- numberOfElements: 현재 쿼리 결과로 조회된 엘리먼트의 개수 

위 내용만 보아도 웹에서 수행할 수 있는 대부분의 페이징 화면을 만드는데 문제가 없을 것이다. 

## Page, Sort 같이 이용하기. 

페이징이 필요한 데이터는 대부분 Page와 Sort 를 함께 이용하게 되어 있다. 

이제는 좀더 실용적으로 2개 모두 활용해 보자. 

Pageable 는 소트 기준을 함께 담을 수 있는 메소드를 제공한다는 것일 이미 알 것이다. 우리는 그것을 이용할 것이다. 

새롭게 메소드를 하나 추가해보자. 

```java
    public Page<UserDto> findAllUserWithPagingAndSort(int page, int sizePerPage, String ascdes) {
        PageRequest pageReq = PageRequest.of(page, sizePerPage, getUserSortByCreatedAtV2(ascdes));
        Page<User> usersWithPage = userRepository.findAll(pageReq);

        log.info("Total Users: " + usersWithPage.getTotalElements());
        log.info("Total Pages: " + usersWithPage.getTotalPages());

        List<UserDto> users = usersWithPage.getContent().stream().map(user -> user.getDTO()).collect(Collectors.toList());

        return new PageImpl<>(users, usersWithPage.getPageable(), usersWithPage.getTotalElements());
    }
```

달라진것은 String ascdes 라는 파라미터가 추가 되었고. PageRequest.of 에서 마지막에 Sort 객체를 전달할 수 있도록 메소드를 호출했다. 

```java
    private Sort getUserSortByCreatedAtV2(String ascdes) {
        if ("DES".equals(ascdes)) {
            return Sort.by("createdAt").descending();
        }
        return Sort.by("createdAt").ascending();
    }
```

메소드는 이전에도 살펴본 소트 메소드 이다. 

### 결과 확인하기. 

```java
Hibernate: 
    select
        user0_.id as id1_5_,
        user0_.birth as birth2_5_,
        user0_.created_at as created_3_5_,
        user0_.name as name4_5_ 
    from
        user user0_ 
    order by
        user0_.created_at desc limit ?,
        ?
```

order by 와 limit 가 정상으로 걸린것을 확인할 수 있다. 

이것 이외에 이전과 동일하다. 

```java
{
  "content": [
    {
      "id": "kido1",
      "name": "kido",
      "birth": "770605",
      "createdAt": "2020-11-16T10:35:37.107",
      "userDetail": {
        "id": "kido1",
        "nick": "kaido",
        "avatarImg": "http://kido.com/img.png",
        "category": "economic",
        "joinedAt": "2020-11-16T10:35:37.107",
        "modifiedAt": "2020-11-16T10:35:37.107"
      },
      "roles": null
    }
  ],
  "pageable": {
    "sort": {
      "unsorted": false,
      "sorted": true,
      "empty": false
    },
    "offset": 2,
    "pageNumber": 1,
    "pageSize": 2,
    "paged": true,
    "unpaged": false
  },
  "last": true,
  "totalElements": 3,
  "totalPages": 2,
  "first": false,
  "size": 2,
  "number": 1,
  "sort": {
    "unsorted": false,
    "sorted": true,
    "empty": false
  },
  "numberOfElements": 1,
  "empty": false
}
```

실행 결과 반환값을 확인하면 pageable.sort.sorted 값이 true 로 되어 있음을 확인할 수 있다. 

sort 실행 결과가 정상으로 수행되었다는 것을 Page 내의 속성값을 통해 확인이 가능하다. 

## Wrap up

지금까지 derived query 를 확인해 보았다. 

간단한 쿼리나, 일반적인 쿼리 대부분은 derived query 로 쉽게 생성이 가능하다.  

또한 페이징과 소팅을 확인해 보았으며, 결과를 어떻게 활용하면 좋을지도 이해할 수 있다. 