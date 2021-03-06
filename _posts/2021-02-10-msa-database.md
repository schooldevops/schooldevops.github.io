---
layout: post
title:  "Microservice Architecture 에서 Database 를 다루는 기술"
date:   2021-02-10 08:45:49 +0900
categories: Architecture
tags: [Architecture, MSA, Monolithic, Database, SAGA]
toc: true
---
# Microservice Architecture Database 

가끔 이런 질문을 받을때가 있다.

"서비스 마다 데이터베이스는 어떻게 구성하셨나요? 컴포넌트별로 분리하실때 데이터베이스도 분리를 하지 않으신거 같은데, MSA 구현에서 데이터 베이스 분리를 안하셨다면 그게 MSA 라고 할 수 있나요?"

마이크로 서비스 아키텍처를 구현할때 서비스별로 데이터베이스 분리는 당연한 것으로 받아 들여진다. 

물론 이렇게 분리하는 것이 대부분 좋은 선택이라고 필자도 생각하고 있다. 그러나 서비스 영역을 어떻게 분리 하느냐에 따라 우리는 다양한 전략을 사용할 필요가 있다. 

첫번째는 도메인 영역에 따라 데이터베이스를 적절히 분리하는 것도 필요하지만, 도메인에 맞게 데이터 베이스를 통합할 필요도 있다. 

두번째는 서비스에 따라 특정에 맞는 최적의 데이터베이스를 선택하는 것 역시 중요하다고 할 수 있다. 

이번 아티클에서는 마이크로 서비스를 구현하기 위한 데이터베이스 선정 전략에 대해서 알아보고자 한다. 언제나 그렇듯이 정답은 없다. 
오직 환경과 조건에 따라 선택지는 다양하며, 선택사항을 이해하고, 다양한 전략을 알아 두는 것은 언제나 정답이라 할 수 있겠다. 

## Monolithic Architecture 에서 데이터를 다루는 방법

모놀리식 아키텍처는 RDBMS의 도입을 기본적으로 고려 대상으로 삼고 있다.  
관계형 데이터베이스는 대부분의 시스템에서 수용할 만한 적절한 성능과 데이터 저장을 위한 다양한 모델을 제공하고 있기 때문이다. 

모놀리식 아키텍처는 데이터 컨시스턴시를 위한 트랜잭션 위주의 사고로 데이터베이스를 바라본다. 
단일 데이터베이스에 다양한 스키마를 정의하며, 제약조건을 통한 데이터 컨시스턴시를 유지하기 위한 모델링을 한다. 

대부분의 서비스들은 서비스들 끼리 상호작용을 한다. 

쇼핑몰 예를 들어보자. 주문 데이터와 고객 데이터, 제품 정보와 제고는 매우 밀접한 관계를 갖고 있다. 고객이 제품을 장바구니에 담고, 주문을 하게 되면 제품의 제고를 검사하고, 제고를 차감하고, 결제를 수행한다. 
주문 테이블과, 제품 재고 테이블, 그리고 결제 테이블은 데이터 일관성을 지키도록 설계 된다. 

모놀리식 아키텍처로 구성된 시스템은 단일 트랜잭션으로 이 모든 데이터의 일관성을 유지할 수 있다. 
그러므로 개발자는 All or Nothing 으로 주문이 성공하면 제품의 재고와 결제 정보, 주문정보는 동시에 일관성 있는 상태로 저장이 된다. 
혹은 이들 중 하나라도 실패한다면, 아무일도 일어나지 않은 초기 상태로 데이터가 유지 된다. 

### 단일 데이터베이스의 장점

- 단일 데이터베이스는 ACID 를 통한 데이터 일관성 유지가 매우 쉽다. 
- 한번의 쿼리를 통해서 CRUD 를 수행하며 하나의 트랜잭션으로 원하는 데이터를 일관성 있는 상태 변경이 가능하다. 
- 데이터 오퍼레이션시 DBMS 고유의 기능을 활용할 수 있으며, 이런 기능들을 이용하여 성능 최적화를 가져 올 수 있다. 

### 단일 데이터베이스의 단점

- 자칫 트랜잭션의 시간을 오래 잡을 수 있다. 
- 동일한 데이터의 다른 컨텍스트(데이터에 대한 이해가 다름) 에서는 데이터를 중복으로 쌓고, 이를 관리하기가 쉽지 않다. 
- 데이터베이스 스키마 변경이 가져오는 사이드 이펙은 컴파일 타엠에서는 파악이 안된다. 런타임에서 발견되는 이슈는 매우 치명적일 수 있다. 
- 시스템의 규모가 커질수록 트랜잭션 볼륨 자체가 증가하며, 분리역시 쉽지 않다. 
- 한번에 많은 조인은 성능 저하의 주범이 되기도 한다. 
- 시스템의 스케일 확장이 가져오는 데이터베이스 커넥션 리소스 부족 이슈가 존재한다.

모롤리식 아키텍처는 일단 기본적으로 단일 데이터베이스를 사용한다. 
이는 작은 시스템에서는 장점으로 작용할 수 있으며, 규모가 커지면서 문제를 일으킬 다양한 변수가 증가함을 의미한다. 
그러나 이해하기 쉽고, 전체 데이터 관계를 한번에 파악할 수 있어, 오히려 관리가 쉽다.

## MSA 는 꼭 서비스마다 데이터 베이스를 가져야 하는가?

처음 던진 화두에서 MSA 는 정말 서비스별로 데이터베이스를 분리해야하는 것일까? 

일반적인 이해는 분리가 좋다라는 것이다. 하지만 분리도 각 서비스마다 분리하는 것은 많은 오버 엔지니어링을 가져온다. 

자 그러면 데이터베이스 관점에서 살펴 보자. 특히 가장 기본이 되는 RDBMS 의 입장에서 살펴보면 다음과 같은 고민이 필요하다. 

- 데이터베이스는 스케일 아웃이 근본적으로 어렵거나 불가능하다. RDBMS는 분산 데이터베이스 구현을 목표로 개발된 시스템이 아니다. 
- RDBMS 는 단일 마스터로 동작한다. 일반적인 시스템 아키텍처는 Active-Standby 로 구성이 된다. 즉, 하나의 마스터만 동작하다가, 문제가 발생되면 Standby 에 복제되어 있던 데이터를 활용하여 마스터 복구를 수행한다. 
- Mysql 의 경우 HA 위해 Multi Master 를 구성 하기도 하며, 이는 평소에는 마스터 한대로 모든 오퍼레이션을 수행하며, 장애가 있을경우 Secondary Master 가 이전에 저널링된 데이터베이스를 저장하고, 자신이 마스터 역할을 수행한다.

위와 같은 특징을 가진 데이터베이스는 근본적으로 취약한 구조를 가지고 있기 때문에 Active-Standby 혹은 멀티마스터 데이터베이스, 마스터 슬레이브 데이터베이스와 같은 백업 시스템을 구성해야한다. 

자 이제 어느정도 감이 올것이다. 서비스마다 데이터베이스는 이상적으로는 좋은 구조이지만, 현실적으로는 매우 많은 오버헤드를 수반한다는 것을 단번에 알 수 있다. 

그러므로 MSA 로 시스템을 구헝할 때 각 서비스마다 데이터베이스를 가지는 구조는 서비스마다 최대 3배의 오버헤드를 수반하게 된다. 우리는 이러한 선택을 쉽게 할 수 있을까? 

모든 시스템은 개발, 운영에 대한 비용을 계산해야하기 때문에 꼭 서비스마다 데이터베이스를 가지는 것은 올바른 선택이 아닐 수 있다. 

가이드를 주다면 비즈니스 도메인(즉, 관심사)마다 데이터베이스를 구성하는 것은 좋은 선택이다. 그러나 처음 우리가 MSA와 Monolithic 을 결정하는 시점에서는 리소스 규모에 따라 신중하게 결정 해야한다.

그렇다면 우린 어떠한 선택지를 가지고 있는지 살펴보자.

## 데이터 베이스 공유 

데이터베이스 공유 시스템은 여러 서비스들이 하나의 데이터베이스를 공유하는 것이다. 

이 구조는 여러가지 형태로 구성이 가능하다. 

- 전체 마이크로 서비스 컴포넌트들이 하나의 DBMS에 엑세스 하여 오퍼레이션을 수행하는 방법이 있을 수 있다. 이 방법은 하나의 DBMS만을 관리해야하기 때문에 관리적인 측면은 적다. 그리고 대부분의 연관 서비스들에 대한 트랜잭션과 조인이 이루어 지므로 가장 편리하게 데이터베이스를 관리할 수 있다는 장점이 있다. 
반면 컴포넌트의 개수가 늘어날 수록 데이터베이스에 가해지는 부하가 증가하며, Domain Driven 관점의 설계나, 이벤트 기반의 아키텍팅과는 다르게 데이터 관점의 아키텍팅으로 흘러갈 공산이 크다. 즉 거대한 모놀리식 구조의 마이크로 서비스 처럼 보이는 아키텍처가 될 수 있다는 점이다. 
물론 시스템이 커질수록 상호 간섭은 더 늘어날 것이고, 이러한 간섭을 분리하기 위해서는 많은 노력이 필요로 한다. 
- 서비스 그룹 혹은 도메인별 데이터베이스 공유와 분리가 있다. 이 방법은 데이터베이스 공유 관점에서는 훌륭한 방법이라고 생각이 든다. 관련 있는 서비스들끼리 데이터베이스를 공유하며, 트랜잭션 역시 공유한다면, 단일 데이터베이스의 단점을 많은부분 보완할 수 있다. 

데이터베이스를 공유하는 구조의 설계는 비즈니스 도메인을 어떻게 잡고, 누가 관리하느냐에 따라 달라질 수 있기 때문에 이러한 RnR 분리는 비즈니스 관점과 확정성을 고려한 설계에 주의를 귀울여야 한다. 

## 동일 DBMS 와 계정별 권한 부여 

마이크로 서비스 아키텍처가 근본적으로 지향하는 단일 서비스 단일 데이터베이스를 구현하고자 하지만, DBMS의 관리 포인트 증가가 우려되는 경우 단일 DBMS를 사용하고 멀티 Account 를 통한 데이터베이스 분리도 고려해볼만한 사항이다. 

이런 구조의 장점은 DBMS 관리 리소스를 극적으로 줄일 수 있으며, 마이크로 서비스가 확장될 때에도, 서비스를 분리가 매우 쉽다는 장점이 있다.

사용자 엔드포인트와 접점인 부분은 분리된 데이터베이스로 동작하며, Batch 나 시스템 데이터의 aggregation 이 필요한경우 여러 스키마에 접근할 수 있는 account를 사용함으로 해서 단일 조인의 이점을 가져올 수 있기도 한 구조가 된다. 

필자의 경우 초기 MSA 설계를 비즈니스 도메인의 성격에 따라 DBMS account 를 생성하고, 접근 권한을 각 서비스 도메인별로 부여했다. 그리고 유사한 비즈니스 도메인 그룹이 필요한 스키마들끼리 그루핑하여 권한을 부여하였다. 이런 구조는 같은 이점을 얻을 수 있었다. 

- 분리가 필요한 코어 서비스는 자신에게 부여된 계정을 통해 다른 계정과 독립적으로 스키마를 사용해서, 데이터분리가 필요한경우 이슈를 최소화 할 수 있었다. 
- 유사한 비즈니스 도메인으로 묶인 서비스들은 조직 구조가 커지지 않은한 변화가 없을 것으로 가정했으며, 이 조직은 실제도 분리되지 않고 통합된 구조로 시스템이 운영되었다. (예. 사용자 계정 정보 - 구매자,판매자를 동일한 계졍으로 두기도 했다)
- Batch와 같이 집계가 필요한경우 어려 스키마를 조회할 수 있는 권한의 account 를 두어 다양한 쿼리를 구하사알 수 있었다. (물론 이 부분은 분리시 고민이 필요한 부분 이라는 것을 인지할 필요는 있다.)

작은 규모의 MSA 라면 동일 DBMS 와 서비스별 계졍을 두는 것은 좋은 선택이다. 

## 서비스마다 자신의 DBMS 가지기 

마이크로 서비스 아키텍처의 최종 종착지는 서비스마다 자신의 DBMS 를 구성하는 것이다. 

처음 이야기 했듯이 기능별로 MSA를 분리하는 방향은 지양하며, 비즈니스 도메인별로 (비즈니스 관심사의 분리) 분리하는 방향으로 설계할 필요가 있다고 했다. 
이러한 비즈니스 영역별로 데이터를 가지는 것은 매우 훌륭한 방법이다. 

보통 주문 결제와 같은 시스템은 데이터 일관성을 위해서 RDBMS 를 선호하며, 실제로도 데이터 정합성 측면에서 단일 트랜잭션으로 묶일 수 있도록 설계하고 구현한다.
그러나 회원 정보의 경우는 RDBMS 를 고집할 이유는 없다. 만약 고객관계 시스템을 만들고, 고객들의 연관 정보들이 중요한 서비스라면 Graph NoSQL 인 Neo4J 등과 같은 DB를 이용하는 것이 현명할 수 있다. 
때로는 CQRS 와 같은 구조를 위해서 RDBMS와 NoSQL 을 동시에 필요한 아키텍처를 도입 할 수도 있다. 

이런 관점에서 볼 때에는 서비스별 DBMS는 궁극적으로 각 특성에 맞는 최적의 데이터베이스를 취할 수 있는 아키텍처라고 할 수 있을 것이다. 

## 트랜잭션 

데이터베이스의 궁극적인 목적은 우리가 원하는 데이터를 저장하고, 원하는 상태를 유지하는 것이다. 
이러한 일관된 데이터를 유지하는 상태를 데이터 컨시스턴시라고 부르며, 이러한 컨시스턴시를 유지하기 위한 방법을 고민한다. 

전통적인 데이터베이스시스템에는 트랜잭션이라는 방법을 통해서 이러한 컨시스턴시를 유지한다. 
이러한 트랜잭션은 단일 시스템에서만 동작한다. 마이크로 서비스와 같이 분리된 시스템, 분리된 데이터베이스를 갖는 환경에서는 트랜잭션을 유지하기가 매우 어렵다. 

모놀리식 아키텍처에서는 컨시스턴시를 유지하는 것이 그리 어려운 일은 아니다.(실제로는 어렵고, 중요한 부분이다. 여기서는 분산 환경에 비해 상대적으로 쉽다는 의미이다.) 
실제는 모놀리식에서도 컨시스턴시 유지는 쉽지 않다. 트랜잭션은 항상 어려운 과제이다. 

그러한데 MSA 에서는 얼마나 어렵겠는가? 분산 환경에서 트랜잭션이라는 것이 실제적으로 존재하는 것일까?
어떻게 분산 환경에서 트랜잭션을 구현 할 수 있는지 알아보자.

### 데이터 컨시스턴시 

데이터 일관성 유지라는 관점에서, 특히나 MSA 환경에서 이를 실현하기 위해 고려해야 할 사항이 몇가지 있다. 

첫번째는 마스터 데이터 상태에 따라, 연관된 데이터의 상태를 의미론적으로 유의미하게 유지 해야한다는 것이다. 즉 마스터 데이터가 삭제라고 되었다면, 나머지 연관된 데이터 테이블의 내용도 삭제와 동일한 상태를 유지해야한다는 것이다. 
두번째는 데이터의 멱등성이다. 동일한 데이터가 중복으로 쌓이게 된다면 그것은 동일한 데이터인가? 아니면 새로운 이벤트로 발생된 데이터인가를 명확히 구분할 수 있어야 하며, 동일한 데이터라면 중복으로 적재되거나 부족하지 않아야 한다. 이는 동일한 요청이 중복으로 들어온경우라도 데이터는 하나만 존재해야한다는 것이 바로 멱등성이다.

이러한 관점에서 우리는 MSA 에서 데이터 컨시스턴시를 지키기 위한 중요한 포인트를 이해할 수 있다.

연관된 모든 데이터의 상태가 동일한 상태가 되어야 하는 것과, 중복으로 적재되거나, 부족함이 없는 상태가 되어야 한다, 이렇게 데이터 상태를 유지하는 것이 데이터 일관성의 핵심 목표이다. 

### 2PC (2 Pahse Commit)

분산 트랜잭션이라는 용어가 한때 매우 인기가 있었던 적이 있었다. 그 구현방법중에 하나가 2PC 라는 방법이다. 

![http://1.bp.blogspot.com/-sS575_YsGS8/VAEh2o6qPtI/AAAAAAAACTg/F4TYvMRDzgQ/s1600/two-phase-commit.png](http://1.bp.blogspot.com/-sS575_YsGS8/VAEh2o6qPtI/AAAAAAAACTg/F4TYvMRDzgQ/s1600/two-phase-commit.png)

from: http://1.bp.blogspot.com/-sS575_YsGS8/VAEh2o6qPtI/AAAAAAAACTg/F4TYvMRDzgQ/s1600/two-phase-commit.png

2PC 는 트랜잭션에 묶여있는 분산 JOB을 위해서 메인 상태가 Pending 으로 변경이 되면서 연관된 서비스에 이벤트 요청을 보내 상태 변경을 명령한다. 
그리고 각 서비스들은 해당 명령을 받은후에 자신의 데이터의 상태를 변경하고, 상태 변경의 결과를 메인 루틴으로 성공/실패를 전달한다. N개의 서비스에 요청을 보냈다면 N개의 응답이 모두 정상으로 오는 경우 마스터 데이터는 커밋이 되며, 다시 각 서비스들에 커밋을 명령하는 구조가 2PC 의 동작 방식이다. 

![http://4.bp.blogspot.com/-l1cZdynJsOg/VAEuw1g5mgI/AAAAAAAACT8/zs2UYwxGjVY/s1600/two-phase-commit-failure.png](http://4.bp.blogspot.com/-l1cZdynJsOg/VAEuw1g5mgI/AAAAAAAACT8/zs2UYwxGjVY/s1600/two-phase-commit-failure.png)

from: http://4.bp.blogspot.com/-l1cZdynJsOg/VAEuw1g5mgI/AAAAAAAACT8/zs2UYwxGjVY/s1600/two-phase-commit-failure.png

처음 이 방식이 나왔을때 분산 트랜잭션의 획기적인 방법처럼 느겨졌었다 그러나 이는 몇번의 실험으로 매우 구현하기가 어렵고, 부수적인 작업이 더 많고, 이것이 새로운 이슈를 생산 하는것을 깨닫게 되었다. 

#### 2PC의 장점

2PC 는 긍정적인 시나리오에서만 장점이 있다. 
즉, 데이터 일관성 측면에서는 2PC는 가장 확실한 방법이었다. 마스터의 상태와 연관 데이터의 상태가 완벽히 일치하는 상황을 염두에 두고 만들었기 때문이다. 
정상적으로만 동작하면 모든 상태는 트랜잭션에 의해서 단일 시스템의 트랜젹션을 그대로 구현할 수 있는 기술이다. 

#### 2PC의 단점 

2PC 는 이론적으로 괞찮은 방법이다. 필자가 괞찮다고 하는 이유는 연관된 서비스 개수가 증가하면 할 수록 더욱 일관성을 유지하기 어려운 환경에 놓인다는 점이다. 
그리고 2PC는 하나의 시스템에서 소유하는 트랜잭션을 너무 오래 점유한다는 문제도 존재한다. 각 서비스들의 트랜잭션 시작포인트 부터 커밋이 완료되는 종료지점까지 고려해야할 사항과 지연된는 시간은 현실적으로 규정하기 너무 어렵다는 문제가 존재한다. 
이렇게 되면, 대용량 트래픽을 수용하기위해서는 2PC는 사용할 수 없다는 문제가 생긴다. (트랜잭션의 점유는 시스템 장애에서 가장 자주 나오는 패턴임을 이해하기 바란다.)
민액 히나의 서비스에서 트랜잭션 커밋을 수행하는동안 이슈가 생긴다면? 어떤 방법을 사용 해야하는가? 라는 문제도 직면하게 된다. 

결국 현실적인 엔터프라이즈 아키텍처에서 2PC는 사용할 수 없는 기술이 되고 말았다. 

### SAGA Pattern

트랜잭션은 일종의 제약조건이다. 분산환경에서 제약조건은 그 조건을 모두 맞추기가 어려운 환경임을 이해하게 되었고, 이에 대한 대응 방안이 필요하면서 SAGA 패턴이 등장하게 된다. 
SAGA 패턴은 결과적인 일관성을 구현하기 위한 하나의 패턴이다. 즉, 필요한 서비스들의 성공이 모두 완료가 되면 결국에는 성공으로 처리가 되며, 하나라도 실패가 되면, 당당 모든 것을 원래대로 돌리지는 못하더라도, 최종적으로는 원하는 상태가 되도록 설계하는 것이다. 
일반적인 MSA 에서는 이러한 SAGA 패턴을 이용하여 일관성을 유지한다. 그럼 어떻게 이런 일관성을 유지하는지 알아보자. 

### SAGA? 

우선 SAGA 부터 알아보자. SAGA 는 이벤트의 시퀀스라고 정의한다. 혹은 트랜잭션 관점에서는 로컬 트랜잭션의 시퀀스 다발을 SAGA라고 한다.

이벤트의 시퀀스 (일종의 배열) 이라고 한다면 다음과 같이 이야기 할 수 있다. 어떠한 모듈에서 다음과 같은 일을 한다고 하자. 

1. A 서비스에 특정 데이터의 상태를 완료로 변경요청 이벤트를 보낸다. (이벤트)
2. B 서비스에 특정 데이터의 상태를 종료로 변경요청 이벤트를 보낸다. (이벤트)
3. C 서비스에 특정 데이터의 상태를 삭제로 변경요청 이벤트를 보낸다. (이벤트)

위와 같이 A에 보내는 이벤트, B에 보내는 이벤트, C에 보내는 이벤트 등과 같이 일련의 이벤트를 시퀀스로 관리하는 것을 SAGA 라고 부른다.

각 이벤트는 응답을 보내게 되며, 응답이 완료되면 다음 이벤트를 보내고, 응답이 성공이면 다음으로, 실패면 보상 이벤트로 각 서비스에 롤백을 보내는 방향으로 처리를 한다. 

이러한 SAGA 패턴을 이용하면 정상 케이스에서는 최종적으로 상태의 일관성을 유지할 수 있게 되며, SAGA 시퀀스 내부에서 하나라도 실패하면 실패 지점 이전의 서비스들에 보상 이벤트를 발생시켜서 초기 상태로 롤백할 수 있도록 하여 결과적 일관성을 보장하도록 하는 패턴이다. 

### SAGA 패턴의 종류 

SAGA 패턴의 방볍은 몇가지 방법들이 존재한다. 

하나는 요청하는 서비스와 요청을 받는 각각의 주체들이 상호작용하는 코리오그라피 방식의 패턴과 요청하는 쪽에서 SAGA 시퀀스를 생성하고, 이들의 처리가 끝나면 다음처리를 처리하도록 조정하는 오케스트레이션 패턴이 있다. 

#### 코리오 그라피 (Chorigraphy based SAGA)

코리오 그라피는 다음 그림과 깉이 설명할 수 있다. 

![Chorigraphy based SAGA](https://i0.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-09-at-6.13.39-PM.png?resize=768%2C817&ssl=1)

from : https://i0.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-09-at-6.13.39-PM.png?resize=768%2C817&ssl=1

즉 로컬 트랜잭션이 이벤트를 발생하면, 이에 대응하는 서비스가 이를 받아 자신의 상태 변경을 수행하고, 다시 연관된 서비스에 이벤트를 던지는 형식으로 트랜잭션을 수행한다. 
이벤트의 진행에서 이슈가 발생하면 보상 이벤트가 발생이 되며, 보상이벤트에 따라 각 서비스는 롤백을 수행하는 구조이다. 

![Chorigraphy based SAGA rollback](https://i1.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-09-at-6.36.17-PM.png?resize=768%2C526&ssl=1)

from : https://i1.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-09-at-6.36.17-PM.png?resize=768%2C526&ssl=1
#### 오케스트레이션 (Ochestration based SAGA)

오케스터레이션은 오케스트레이터 가 트랜잭션 순서에 따라 서비스를 호출하고, 하나의 서비스가 성공하면 다음 서비스를 호출하는 순서로 오케스트레이션을 수행한다. 
특정 서비스의 응답이 실패가 된다면 오케스터레이터는 다시 보상 트랜잭션을 서비스로 호출하는 구조로 동작한다. 

![https://i0.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-11-at-7.41.06-PM.png?resize=768%2C489&ssl=1](https://i0.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-11-at-7.41.06-PM.png?resize=768%2C489&ssl=1)

from : https://i0.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-11-at-7.41.06-PM.png?resize=768%2C489&ssl=1

## 최적의 데이터베이스 아키텍처 선택하기. 

지금까지 MSA 데이터베이스를 구성하는 다양한 방법에 대해서 살펴 보았다. 

데이터를 서비스 혹은 도메인마다 분리를 할지, 통합된 공유 데이터베이스를 이용할지에 대한 관점을 살펴 보았고, 
서비스에 맞는 데이터베이스 종류에 따라 무엇을 이용할지를 가늠해 볼 수 있었다. 

어디까지나 시스템의 환경과 조직의 환경에 맞는 구조를 선택해야하며, 언급한 것과 같이 선택의 기준을 여러가지 팩트들을 바탕으로 최적의 구조를 가져가는 것이 필요하다고 할 것이다. 

처음 MSA 를 시작하는 작은 조직에서라면, 관리할 수 있는 범위 내에서 비즈니스 도메인을 분리하고, 각각의 데이터베이스를 구성하기 보다는 하나의 데이터베이스에서 계정만을 분리하여 시스템을 구성하는 것이 좋은 시작점이라고 할 수 있다.

그리고 규모가 커지게 되면, 혹은 처음부터 각각의 업무 도메인으로 조직이 구성되었다면 도메인에 따른 데이터 분리가 최적의 방법일 것이다. 

무엇보다 MSA 는 서비스의 특징에 맞는 다양한 데이터베이스를 선택할 수 있도록 서비스 관점에서 니즈를 살펴 보는것도 매우 중요한 포인트이다. 만약 하나의 업무 도메인에 몇개의 서비스가 있고, 데이터베이스 종류도 다른것을 사용한다면, 서비스를 미리 컴포넌트로 분리해두는 것이 좋은 선택일 것이다. 

최종적으로 말하고 싶은것은, 리소스의 규모와 관리 가능한 정도에서 데이터베이스도 선택하라는 것이 필자의 주장이다. 
단순한 구조에서 좀더 복잡한 구조로 전개해 나가는 벙법을 고려하자는 것이다.

## 서비스별 데이터 베이스 분리 기술 

자 지금까지 데이터를 어떻게 접근하고 관리하는가에 대해서 알아 보았다면 이제는 모놀리식 아키텍처에서 마이크로 서비스 아키텍처로 전환할때 데이터를 분리하는 방법을 살펴 보자. 

필자의 경험으로 듀얼 라이트 방식을 권장하며 순서는 다음과 같다. 

1. 데이터베이스와 스키마를 별도로 구성한다. 
2. 분리하고자 하는 모듈중에서 데이터베이스와 연관된 모듈 혹은 좀더 큰 범위로 서비스를 분리한 모듈을 기존 코드에서 떼어 구현한다. 
3. 상위 로직에서 Old/New 양쪽으로 요청을 보낸다. 이때 기존 모듈의 로직은 그대로 두고, 새로만든 로직의 경우 에외 처리를 수행하여 트랜잭션이 롤백이 되지 않도록 구현한다. (즉, 기존 로직에 영향을 가지 않도록 대응방안을 마련한다.)
4. 기존 데이터와, 신규 데이터를 동일한 트랜잭션 내에서 상호 비교한다. (정상적이라면 분리준비가 완료된 것이다.)
5. 신규 데이터가 쌓이기 이전의 과거 데이터들을 신규 데이터로 이관 시킨다. 
6. 기전 로직의 플로우를 끊어내고, 새로운 로직만 타도록 코드를 수정 배포하여 분리 작업을 완료한다. 

데이터를 분리해 내기 위한 방법중에 가장 좋은 방법은 데이터의 구조를 변경하지 않고, 로직의 흐름만 양갈래로 분리하고 동일한 데이터를 저장하는 것이다. 
그리고 데이터를 비교하여 차이가 없다면 기존 데이터를 분리해도 이슈가 없을 것이다. 

데이터를 분리하는데 이것 이외에도 다양한 방법이 있을 수 있지만 듀얼라이트와 같은 방법이 경험적으로 가장 안전한 방법이었다. 시스템의 중요도에 따라서 다양한 전략을 사용하기 바란다. 

## 결론 

지금까지 MSA 에서 데이터베이스를 분리하는 몇가지 전략을 알아 보았다. 

MSA라고 해서 DB를 완전히 서비스마다 분리하는 것만이 능사는 아니며, 필요한 경우에는 그룹핑하여 DBMS를 하나로 가져가는 방법, 동일한 DBMS에 계정으로 분리하는 방법도 좋은 선택일 수 있다. 
어디까지나 조직의 상황에 맞는 MSA 뷴리 규모를 선택하고, 서비스마다 최적의 DBMS를 선택하고, 듀얼라이트와 같은 검증가능한 방법을 도입할 수 있도록 다방면의 접근 방법을 검토하는 것이 필요하다. 

분산 환경에서 트랜잭션 보장을 위한 방법을 도입하고, 보상트랜잭션을 구현하는 등의 작업은 필수 조건이라고 할 수 있을 것이다. 
현명한 결정으로 성공적인 MSA 전환이 될 수 있는 가이드로 봐주면 좋을듯 하다. 


## 참고

[https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part](https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part/)

[https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part-2/](https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part-2/)

[https://microservices.io/patterns/data/database-per-service.html](https://microservices.io/patterns/data/database-per-service.html)

[https://dzone.com/articles/two-phase-commit-distributed](https://dzone.com/articles/two-phase-commit-distributed)