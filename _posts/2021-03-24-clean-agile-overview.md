---
layout: post
title:  "[IT 책읽기] Clean Agile Overview"
date:   2021-03-24 11:45:49 +0900
categories: Agile
tags: [CleanAgile, UncleBob, Agile]
toc: true
---

# Celan Agile 책읽기 

Clean Agile 은 로버트.C.마틴 의 가장 최근 책으로 Agile 이라는 단어를 탄생시킨 멤버중 한명이다.

평소에 존경하는 분으로 엉클밥으로 알려져 있으며, 그의 한마디 한마디가 마음에 꽃히는 것 같다. 

Clean Agile 책 읽기를 시작하면서, 새겨 들어야할 사항들을 박제해 두어야 겠다. 

## Agile 개요

### Agile Manifesto

- 공정과 도구보다 개인과 상호작용
- 포괄적인 문서보다 작동하는 소프트웨어
- 개약 협상보다 고객과의 협약
- 계획을 따르기보다 변화에 대응하기

### 철십자. 

  - 프로젝트의 근본 물리 법칙이 존재
  - 좋음, 빠름, 저렴함, 완성 이라는 4가지 축이 있으며, 우리는 이중 3가지만을 선택할 수 있다.
  - 프로젝트 관리자는 이 철십자의 비중을 관리하는 것 (4가지 모두 100% 를 채울 수는 없다.)

### Data

  - Agile 에서 중요한 것은 데이터이다. 
  - 중요 데이터
    - 팀의 속도: 스프린트가 0 ~ 3, 4 를 진행하면서 팀이 수행할 수 있는 평균 포인트를 확인할 수 있다. 이것으로 팀의 속도를 가늠할 수 있다. 
    - 번다운 차트: 현재 팀의 속도를 기준으로 남아있는 일이 언제 끝이날지 가늠하고, 실제 최종적으로 언제 프로젝트가 완료 될지를 가늠할 수 있다. 

## 프로젝트에서 조정 할 수 있는 값

- 가장 먼저 아는것
  - 마감 기한이 정해져 있다는 것.
  - 마감 기한은 비즈니스의 결정사항으로 대부분 변경되지 않는다. 
- 변경이 가능한것
  - 요구사항
  - 요구사항은 언제나 변경된다. 
  - 요구하는 이해 관계자들도 자신이 무엇을 원하는지 알지 못한다. 

## Warterfall 방법

- 정해진 마감일정
- 분석 > 설계 > 구현 > 죽음의 행진 > 오픈 (저품질, 고객이 원하는 그림이 아닌 제품)
- 분석, 설계
  - 잘된 분석, 잘된 설계 기준이 없다. 
  - 분석 시간은 예상한 시간이 완료되면 그냥 지나간다. 
  - 완료된 것이라고 생각하며
- 구현
  - 구현은 실제로 만들어 내는 행위
  - 명확한 기준이 존재하며, 이 기준에서 분석, 설계가 잘못되었는지 파악이 됨
  - 다시 분석, 설계로 갈 시간적 여유가 없으므로, 변경되는 요구사항을 무작정 땜질하면서 개발이 진행이 됨.
  - 결국 (죽음의 행진) 으로 모두가 뛰어들어, 마감을 향해 달려나감

## Agile 방법 

- 정해진 마감일정
- 스프린트 0: 해야할일을 스토리로 분리하고 몇개의 스토리를 진행해 보는 것 
  - 첫번째 데이터( 스프린트 0에서 수행된 팀의 속도가 데이터를 확인하는 단계)
- 스프린트 1 ~ 4: 스프린트 기간동안 스토리를 몇 포인트까지 진행하는지 데이터를 누적하며 쌓아가는 단계 
  - 마감 기한내 프로젝트를 완료 할 수 없다는 것을 데이터로 확인하는 단계
- 희망은 버리고 관리하는 단계
  - 마감기간 내 완료 할 수 있다는 희망 대신 춥고 힘든 현실을 초기부터 끊임없이 보여주는 단계
  - 애자일은 빠르게 나아가는 것이 아니라. 올바르게 나아가는것
  - 현타를 맞아야 관리를 할 수 있다. 


## 철십자 관리하기. 

- 좋음, 빠름, 저렴한, 완성 이 4가지 축에서 비중을 관리해야한다. 

## 일정 변경하기

- 비즈니스 의사 결정으로 인해서 일정은 변경하기는 거의 불가능하거나, 어렵다.
- 가능성이 있는것은 첫번째 목표 일정 대안으로 다른 일정이 있는 경우에만 가능성이 있다. (희박)

## 사람 추가하기

- 브룩스의 법칙: 지연된 프로젝트에 사람을 추가하면 더 늦어진다. 
  - 학습곡선 + 학습에 필요한 기존 인력의 시간 소비 = 결국 더 늦은 완료일 
- 추가 인력에 대한 비용이 더 들어간다. 

## 품질 떨어 뜨리기

- 쓰레기같은 제품으로 품질을 낮추면 시간은 빨라 지는가? 결국 빨라지지 않는다. 
  - 코드리뷰 없이, 테스트 코드 없이, 리팩토링도 하지 않고 --> 결국 쓰레기 코드가 양산하는 오류가 발목을 잡음
- "빠르게 가는 유일한 방법은 제대로 가는 것이다."

## 범위 변경하기. 

- 지정된 목표일가지 꼭 하지 않아도 되는 일은 반드시 존재한다. 
- 그러나 고객은 합의하지 않을 것이다. 
  - 스프린트에서 수행한 데이터를 제시하여 현실을 논리적으로 제시한다. 
  - "합리적인 조직은 데이터가 이긴다."

## 비즈니스 가치 순서

- 범위를 줄이는 방법은 비즈니스 가치 순서대로 일을 하는것
- 가장 중요한 것을 이해 관계자와 협의해서 그것부터 진행하는 것이 필요없는 것을 찾고 제거하는데 결정적임

## XP 방법론

XP 방법론읜 Agile 중 가장 잘 정리되어 있고, 가장 완전, 가장 덜 혼란스러운 방법론

사실상 다른 모든 애자일 프로세스는 XP의 부분 집합 혹은 XP의 변형 

### XP Lifecycle

- 계획 게임(Planning Game)
  - 프로젝트를 기능, 스토리, 작업으로 분리
  - 크기 추정, 우선순위 결정
  - 기능, 스토리, 작업의 일정 정리
- 작은 릴리즈(Small Release)
  - 팀의 일을 작은 단위로 분리
  - 작은 단위로 릴리즈 
- 전체 팀(Whole Team)
  - 게빌 팀이 여러가지 역할로 구성됨
  - 프로그래머, 테스터, 관리자 등이 같은 목표를 위해 함께 협동하는 것
- 지속 가능한 속도(Sustainable Pace)
  - 개발팀이 번아웃 되지 않도록 관리하는 것
  - 프로젝트는 마라톤이다. 
- 공동 소유(Collective Ownership)
  - 사일로를 막음
  - 모든 것은 공유하고 공표하도록 한다. 
- 지속적 통합(Continuous Integration)
  - 피드백 고리가 자주 돌게 만드는데 집중
  - 이를 통해 팀이 프로젝트중 현재 위치를 파악하도록 해줌
- 메타포(Metaphor)
  - 시스템에 대해서 팀과 사업 부서가 의사 소통할 때 사용하는 어휘나 표현을 만들고 공유
- 짝 프로그래밍(Pair Programming)
  - 기술 팀이 지식을 공유, 상호 리뷰, 협력
  - 헌신과 정확성을 극대화
- 단순한 설계(Simple Design)
  - 팀이 노력을 낭비하지 않도록 도와주는 방법
- 리팩토링 (Refactoring)
  - 모든 작업 산출물의 지속적인 개선, 향상을 장려
- 테스트 주도 개발 (Test Driven Development)
  - 기술팀이 최고의 품질을 유지하면서도 빠르게 움직이기 위해 사용하는 안전망

## Agile Manifesto 와 XP

### 공정과 도구보다 개인과 상호작용

- 전체팀, 메타포, 공동소유, 짝 프로그래밍, 지속 가능한 속도

### 포괄적 문서보다 작동하는 소프트웨어

- 인수테스트, 테스트 주도 개발, 단순한 설계, 리팩토링, 지속적 통합

### 계약 협상보다 고객과의 협력

- 작은 릴리즈, 계획 계임, 인수 테스트, 메타포

### 계확을 따르기보다 변화에 대응하기

- 작은 릴리즈, 계획 게임, 지속 가능한 속도, 테스트 주도 개발, 리펙토링, 인수 테스트 

## 결론

Clean Agile 의 개요만 정리해 보았지만, Agile 의 가장 중요한 포인트를 짚고 있는 것 같다. 

Agile Manifesto 는 Agile 을 표현하는 가장 함축적이면서도 기본을 보여주며, 프로젝트에서 무엇을 우리가 바꿀 수 있는지, 그리고 어떻게 그것이 Agile 을 통해서 만들어 지는지를 알 수 있게 되었다. 