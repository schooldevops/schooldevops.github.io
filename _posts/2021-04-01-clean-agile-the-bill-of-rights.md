---
layout: post
title:  "[IT 책읽기] Clean Agile 권리장전"
date:   2021-04-01 08:45:49 +0900
categories: Agile
tags: [CleanAgile, UncleBob, Agile, Rights]
toc: true
---

# Celan Agile 책읽기 (권리장전)

## The Bill Of Rights (권리장전) 

언제나 그렇지만 Uncle Bob의 통찰력과 언어 선택 능력은 탁월하다. 

권리장전 이라는 단어를 보고 문뜩, 고객과 개발자가 이런 권리장전을 기준으로 일을 하게 된다면 얼마나 더 좋아질까? 라는 생각이 들었다. 

이런 단어가 너무 반갑기도 하고, 획기적이라고 생각하는 이유는 아마도, 우리 (고객, 개발자) 모두가 자신이 요구할 권리와 당연해 해 주어야할 일을 올바로 하지 못하는 것은 아닐까 라는 생각이 든다. 

## 고객 권리장전

### 전체적인 계획을 알 권리가 있다. 무엇을, 언제, 얼마의 비용으로 완성할 수 있는지 알 권리가 있다.

- 고객은 일정과 비용을 초기에 산정한다.
- 그리고 이 비용과 일정에 원하는 제품을 개발할 수 있을지 질문한다. 
- 확실하고 정밀한 계획이 필요하다. 이를 위해서 불확실성을 관리해야하며, 계획 추정 일정이 어느정도 불확실한지 적절히 기술하고, 대처하기 위한 수단 강구해아한다.
- 추정을 위한 초기 스프린트와 팀의 속도를 고려한 데이터에 기반한 추정이 필요하다. 

#### 이해 해보기

- 전체적인 계획 이라는 것에서, 고객이 원하는 일정을 100% 지킬 수 있는 보장은 없다.
- 일정과 비용만 나와 있기 때문에 어느정도 퀄리티와 정확한 제품을 아직 정의하지 않았기 때문이다.
- 분석, 설계, 구현 과정을 진행하면서, 그리고 반복수행을 통해서 그 정확도는 향상되어 가는 것이며, 고객과 지속적인 피드백으로 지정된 비용과 일정에 합당한 제품을 구체화 하는 과정이 필요할 것이다. 
- 즉, 고객 권리장전의 현재 부분을 해결할 수 있는 것은 Agile Manifesto 의 "개약 협상보다 고객과의 협약" 일 것이다. 

### 반복 주기마다 가능한 한 많은 가치를 얻을 권리가 있다. 

- 애자일에서는 반복 주기라는 지정된 주기단위로 개발을 진행한다. 
- 반복주기가 끝날때마다, 고객에게 가장 가치 있는 스토리(일감단위) 를 전달해야한다. 
- 그러기 위해서는 반복 주기를 시작할때, 가장 높은 우선순위를 결정하는 거이 중요하다. 

#### 이해 해보기

- 반복은 보통 스프린트라고 부른다. (스크럼 방법론에서는 스프린트가 공식 용어이다.)
- 스프린트 시작은 스프린트 플래닝을 한다. 이때 가장 가치가 높은 우선순위의 스토리를 스프린트 기간동안 수행하는 것이다. 
- 스토리가 확정되고, 할당이 되면 모두가 합의한 것이며, 우리는 이 합의릴 지키기 위해서 스프린트 (그야말로 최선을 다해서 달려야한다.)를 수행한다.
- 스프린트가 끝이 나면, 리뷰를 하며, 이때 우리가 만든 스토리가 제품이 되며, 고객에게 제품으로 제공이 되어야한다. 
  - 제품으로 제공이라는 의미는 동작하는 제품이며, 테스트가 끝이난 완전한 작은 제품단위어야 한다. 
- 회고를 수행하면서, 팀의 속도를 추산하고, 약속의 이행에 대한 장애물을 점검하고 피드백을 한다. 
  - 사실 이 과정이 어쩌면 가장 중요하다고 할 수 있다. 
  - 이 과정에서 데이터를 분석하고, 올바른 방향으로 가기 위한 모든 것이 이 회고의 결과에서 발견되기 때문이다. 
- MVP(Minimum Viable Product) 이 권리장전의 현재 부분에서 가장 중요한 요소라고 생각이 든다. 가치를 측정하는 것은 실제 전달된 제품에서 나오는 것
  - 여기서 Minimum(최소한)의 의미는 작다의 의마가 이니다. 이것은 핵심, 가장 가치가 있는의 의미이다. 

### 작동하는 시스템을 통해 진척도를 알 권리가 있다. 개발자는 고객이 제공한 테스트로 계속해서 시스템을 검사하여 동작을 증명해야한다. 

- 고객은 중간 진척도를 즉각 확인할 수 있어야 한다. 
- 이는 동작하는 시스템을 통해서 확인이 가능하며, 고객은 진행상황을 확인하기 위해서 조건을 지정할 권리를 갖는다. 
- 그리고 이 조건에 맞는지 언제든지 빠르게 확인이 되어야 한다. 

#### 이해 해보기

- MVP 와 같다, 최소한의 동작하는 제품 (이전에 설명한 내용과 같다.)
- 작동하는 시스템은 스프린트 과정에서 계속해서 유지 되어야 하며, 고객은 이 제품들의 기능과 조건들이 완료된 것을 보고 전체 제품중 어디까지 만들어 졌는지를 파악하게 된다.

### 과도한 비용 추가 없이 기능이나 우선순위를 바꿀 권리가 있다.

- 소프트웨어(쉽게 바꿀 수 있는 것) 의 의미대로 쉽게 변경이 되어야한다. 
- 고객의 요구는 당연한 것이다. 

#### 이해 해보기

- 고객의 요구사항은 수시로 바뀐다.
- 유명한 말이 있다. "고객은 자신이 무엇을 원하는지 모른다." 
  - 고객은 현재까지 제품을 보고 자신이 몰랐던 가치를 발견할 수도 있을 것이다.
  - 고객은 자신이 생각했던 제품의 모습과 실제 제품의 가치가 다르다는 것을 나중에야 파악할 수도 있다. 
- 우리가 제품을 만드는 이유는 고객에게 올바른 제품을 전달하는 것이기 때문에 우선순위를 변경하는 것을 당연한 것이라 받아들여야한다. 
- 단 전제 조건은 과도한 비용 추가 없이 이며, 이것을 명확하게 고객이 인지할 수 있도록 데이터를 제공해 줄 필요도 있다. 

### 일정이나 추정이 바뀐 경우 제때 알림을 받고, 목표 일자에 맞추기 위하여 업무 범위를 어떻게 줄일지 결정할 권리가 있다. 언제든지 프로젝트를 취소 할 수 있다. 이 경우 해당 시점까지 지불한 비용이 반영된 유용한 시스템을 받을 수 있다. 

- 고객은 일정에 맞출 것을 요구할 권리가 없음을 주지해야한다. 
- 고객의 권리는 업무 범위를 바꾸어 일정을 관리하는 것으로 한정된다. 
- 일정이 변경될 것을 미리 고객이 알 수 있도록 정보를 제공해야한다. 

#### 이해 해보기

- 올바른 추정을 시작부터 할 수는 없다. 
- 그러나 추정을 정확하게 맞추어 가는 과정이 필요하며, 고객에 일정의 산출 근거를 제시할 필요가 있다. 
- 매 스프린트마다 의사결정이 필요하다. 이 의사결정을 위한 정보를 제공 고객이 원하는 것이다. 
  - 이것이 최종 제품의 크기를 결정할 수 도 있고, 프로젝트를 접을 수도 있는 것이다. 

## 개발자 권리장전 

### 명확하게 정의된 우선순위와 함께 무엇이 필요한지를 알 권리가 있다. 

- 개발자는 요구사항과 요구사항의 중요도를 정확히 따질 권리가 있는 사람
- 일정 추정과 마찬기지로 요구사항도 당연히 현실적으로라는 제약이 있음
- 이 권리는 반복 주기 범위 내에서만 적용
  - 반복 주기 밖에서는 요구사항, 우선순위 변경
  - 반복 주기 내에서는 요구사항, 우선순위는 고정 
- 작은 부분이라면 일부 반복주기내 요구사항 변경을 허용하기도 함 

#### 이해 해보기

- 우선순위는 가장 중요한 요소이다. 그리고 고객의 니즈를 가장 잘 반영한 것이다. 
- 요구사항은 스프린트 초에 확정이 되며, 확정된 요구사항은 개발자가 최선을 다해서 완성해야한다. (약속이기 때문이다.)
  - 확정된 요구사항과, 필요 사항들에 대해서 꼼꼼히 확인해야, 올바르게 개발이 된다.
  - 대충 추정하게 되면 결국 원하는 가치를 제공할 수 없으므로, 당연한 권리로 확실히 요청해야한다. 


### 언제나 높은 품질의 결과물을 만들 권리가 있다. 

- 가장 심오한 권리, 개발자는 일을 잘 할 권리가 있다. 
- 개발자가 자신의 경력에 오점ㅇ르 남기거나, 직업 윤리를 어기도록 강용할 권리가 없다. 

#### 이해 해보기

- 프로젝트는 보통 6개월에서 1년이 진행이 된다.
- 이 기간동안 개발자 역시 우수한 제품을 만들고, 이력서에 굵은 글씨로 박힐만한 제품을 개발하고자 한다. 
- 저품질을 만드는 것은 고객, 개발자 모두에게 이익이 되어야 한다.

### 동료나 관리자, 고객에게 도움을 요청하고 받을 권리가 있다. 

- 다양한 형태의 도움이 존재
  - 개발자 끼리 문제 해결, 결과 확인, 상호 가르쳐주기 등
  - 고객에게 요구사항을 상세히 설명해달라는 도움요청
  - 우선순위를 명확히 결정해 달라는 요청 
- 도움을 청하는 권리와 도움을 줘야할 의무 함께 오는것

#### 이해 해보기

- 크로스 펑셔널 개발 조직의 중요성을 알 수 있다. 
- 프로젝트는 혼자 하는 것이 아니라 함께 하는것, 그리고 서로 도와가면서 어려움을 극복하고, 문제를 해결해 나가는 것이다. 
- Daily Scrum 미팅, 회고 단계에서 이러한 도움을 구하고, 도움을 줄 있다.
  - Daily Scrum 미팅: 어제한일, 오늘할일, 이슈
    - 조직의 리더, 혹은 함게 하는 동료는 "이슈" 에 집중할 필요가 있다. 
    - 이슈는 우리 모두의 책임이 될 수 있기 때문이다. 팀은 어려움을 함께 헤쳐 나가는 것
  - 회고: KPTA
    - 회고에서 Problem 을 도출하고 관리 되어야한다. 
    - 이것을 사전에 찾고, 해결하지 않으면 프로젝트 끝까지 방해가 된다. 
  
### 자신만의 추정치를 만들고 갱신할 권리가 있다. 

- 업무 추정은 본인이 하는 것
- 새로운 요인으로 추정은 언제나 변할 수 있는 것
- 추측은 시간이 가면서 정확해 지는 것이며, 절대 약속이 아닌 것이다. 

#### 이해 해보기

- 추정치는 근거 있게 해야한다. (대충은 대충의 제품을 만들 뿐이다.)
- 플래닝에서 스토리 점수를 매기고, 일정을 추정한다. 
  - 담당자가 추정하는 것이며, 정직한 추정이 필요하다. 
  - 이 때 도움을 주는 것이 플래닝 포커(플래닝 게임) 이 도움을 준다. 
- 돌인줄 알았는데 암석이 땅속에 숨어 있을 수 있다. 
  - 잘못된 추정은 스프린트 회고에서 반드시 검토하고, 개선해 나가도록 해야하는 중요한 포인트이다. 

### 담당 업무를 할당 받는게 아니라 수락할 권리가 있다. 

- 전문가는 일을 수락하지 할당받지 않는다. 
- 전문 개발자는 모든 일이나 과제에 대해서 "아니오" 라고 말할 권리가 있다. 
  - 과제를 완수할 수 있는 확신이 없을 수 있고
  - 더 적적한 사람이 있어서 그럴수도 있다. 
  - 개인적인 이유나 양심일 수 있다. 
- 수락은 책임을 의미한다. (수락은 약속하는 것이다.)
  - 수락했으면 품질과 실행에 책임을 져야한다.
  - 일정을 추정하고 갱신하며, 팀과 의사소통과 도움요청을 해야한다. 
- 시니어, 주니어 함께 일을 하는 경우에도 일을 맡아달라고 부탁할 수 있지만 강요할 수 없다. 

#### 이해 해보기

- 전문가로써 일을 해야한다. 전문가는 스스로에 대한 확신과 신념이 필요하다. 
  - 수락할지 결정할 수 있으며, 이에 대한 전문가 다운 자세로 일을 해야한다. 
- 스프린트 플래닝에서 일의 재미와 의미를 가질 수 있는 방법
  - 스토리를 자원해서 가져가는 것
  - 자신의 선택이며, 자신이 애정을 가지고 일을 수행해 나가는 방법이다. 
  - 때로는 진짜 하고 싶었지만 못하고 하기 실은 일일 수 있다. 이때에는 팀이 도움을 주고, 도움을 요청할 수 있는 문화가 필요하다. 


## 결론

권리장전 이라는 용어가 참 매력적이다. 권리라는 의미는 누군가에게 요구할 수 있는 것이며, 당연히 그래야하는 내용들로 구성된 문서, 계약, 협약 등이다.

나열된 권리장전 들을 살펴 보면서, 개발자로써 어떤 자세로 일을 해야하는지 한번 돌아볼 수 있게 되었다. 

특히 고객의 권리를 바라볼 때에는 고객과 실랑이를 벌이던 내용들이 어찌보면 당연히 내가 했어야하는 것이었고, 개발자의 권리에서 바라보면 잘못된 추정과 Yes 가 어떠한 결과를 가져오게 되는지도 주마등 처럼 흘러가는 것을 느낀다. 

마지막으로 Agile 이 이래서 재미 있는것 같다. 당연한 것을 새삼 알아가는 느낌 말이다...


