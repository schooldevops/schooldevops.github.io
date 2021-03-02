---
layout: post
title:  "Improve your product quality from mistake and error"
date:   2021-02-18 09:45:49 +0900
categories: Agile
tags: [Agile, fault, error, mistake, pokeyoke, postmorterm, 5why, learn]
toc: true
---

# 실수와 오류로 부터 배우는 품질향상

사람은 누구나 실수를 한다. 사람이기 때문에 실수를 한다. 

작은 실수도 있지만, 매우 큰 치명상을 입는 실수도 있고, 기업에서는 막대한 손실을 가져올 수 있는 실수도 존재한다. 
이런 치명적인 실수를 우리는 사고라고 부르기도 한다. 

특히나 IT 업무 관점에서 볼 때 실수로 인해서 고객에게 불편을 안겨주거나, 심지어는 고객을 잃는 경우도 있다. 

누구나 실수와 사고를 일으킬 수 있기 때문에 이러한 것들을 완젼히 제거하기는 불가능하다. 그러나 한가지 긍정적인 방법은 이런 실수를 다시 발생하지 않도록 대응책을 마련할 수 있다. 

이번 아티클에서는 실수와 장애에 대처하는 방법들에 대해서 알아보고, 어떻게 발전해 나아가야할 지에 대해서 고민해 보고자 한다. 

## 실수와 오류

사전적인 의미로 실수는 사람이 인지하지 못한 일이 일어나는 것으로 왜 발생했는지를 알고 있는 것을 말한다. 반면 오류(error)는 지식의 부족으로 인해서 문제를 일으키는 것을 이야기 한다. 

컴퓨터가 수행하는 일에서 실수는 존재하지 않는다. 그러나 오류는 발생할 수 있다. 사람은 이러한 실수와 오류 둘다 만들어내며, 사람의 실수가 컴퓨터의 오류를 만들어 내기도 한다. 

이렇게 분리한 이유는 실수와 오류가 가지는 특성으로 인해서 대응 방식이 조금 다르기 때문이다. 

실수의 경우에는 실수를 방지하기 위한 방법이 사용되며, 실수를 하지 않게 가이드라인을 제시하거나, 제약조건을 주는 형태로 진행이 될 것이다. 

오류의 경우에는 사전에 미리 알 수 없는 일이 발생하므로, 오류를 검출하고, 오류에 대해서 대응하는 체계를 가지는 방향으로 진행이 될 것이다. 

서로 진행하는 방향은 약간 다를 수 있으나, 중요한 것은 단순히 인지하는 것을 넘어서 실수와 오류를 통해서 배움이 있어야 하며, 이러한 배움을 조직과 시스템에 반영하여 개선하는데 그 목적을 두어야 할 것이다. 

## 품질 

품질을 살펴보자. 깊이있는 내용을 논하기에는 일단 필자의 전문분야가 아니라 더 논할 수 없겠다. 

그러나 최소한 필자의 생각으로 품질은 가격대비 우수한 제품을 말한다고 정의내리고 싶다. 낮은 가격에서도 높은 품질을 유지한다면 제품 가격은 자연히 올라갈 것이고, 높은 가격임에도 품질이 낮다면 제품 가격은 떨어져야하거나, 더이상 팔리지 않게 될 것이다. 

제품을 구매하는 입장이 아니라 제공하는 입장에서도 제품에 들어가는 리소스에 비례해서 품질이 높다, 낮다라는 평가를 받게 될 것이다. IT 제품인 소프트웨어라면 이러한 리소스에 투입된 인력, 자동화, 조직문화, 오류를 대처하는 마인드셋과 실천력 이 모든것 역시 포함된다고 할 수 있다. 

소프트웨어의 품질이 좋다는 것은 보통 이렇게 이야기 할 수 있을 것이다. 

- 빠른 반응속도 (1초 이내 응답)
- 편리한 UX (직관적으로 사용할 수 있으며, 필요한 서비스를 적절한 시점에 유도하는등)
- 고객의 실수를 미연에 방지하는 메커니즘 (잘못된 고객의 실수로 인해서 고객에게 시간, 금전 피해를 주지 않도록 하기)
- 장애없이 사용할 수 있는 소프트웨어 (제품의 이슈로 고객이 제품을 사용할 수 없는 상태가 되지 않도록 고가용성 제공)
- 고객의 니즈를 빠르게 반영하여 업그레이드 되는 제품 (고객의 불편, 니즈를 캐치하여 제품에 반영하는 가시성 있는 로드맵)

위와 같은 측면에서 문제를 정의하고, 진단하며, 빠르게 대응할 수 있는 시스템을 갖출 필요가 있다. 이런 시스템은 한번의 기획으로 완성되지 않으며, 반복되는 이슈와 문제로 부터 배우고, 이를 실제 제품에 반영하는 과정을 통해서 달성 할 수 있다. 

## 장애를 대응하는 기존 방법 

우선 장애를 대응하는 기존 방법부터 알아볼 필요가 있다. 

일반적인 장애 보고서는 다음과 같다. 

- 장애 발생시간 및 종료시간
  - 발생시간: 2020-12-01 11:00
  - 장애인지: 2020-12-01 11:05
  - 장애조치: 2020-12-01 11:20
  - 종료시간: 2020-12-01 11:30
  - 총 리드타임: 30분
- 장애 등급
  - 2등급
  - 일부 고객이 제품을 장바구니에 담지 못하는 현상 
- 장애 영향도 
  - 해당 시간 평균 사용유저수: 10,000명
  - 해당 시간 오류 경험유저수: 100명
  - 일부 사용자가 제품에 장바구니를 30분간 담지 못하는 현상 발생 
- 장애 원인
  - 잘못된 프런트 소스 배포로 인해서 배포 과정에서 잘못된 버젼을 사용한 고객이 장바구니에 제품을 담지 못하는 현상 확인
- 장애 처리상황
  - 롤백후 웹서버 캐싱 초기화로 장애 해결 
- 재발방지 대책
  - 잘못된 소스 코드가 배포 되지 않도록 리뷰강화
  - 스테이지 테스트 강화 

어떻게 보면 매우 쳬계적이며, 모든 것이 다 갖춘 문서이다. 그러나 실제 사용시 변화폭이 매우 크며, 작성 방법도 다양하다. 

### 이 보고서에서 얻을 수 있는 정보 

- 장애 시간이 30분 정도이다.
- 1%의 장애이지만 실제 고객에게 영향을 준 등급이 높은 장애이다.
- 잘못된 소스가 배포된 장애
- 리뷰를 강화해서 재발을 방지하자.
- 스테이지 테스트를 강화하자.

위 정보로 우리가 과연 제품의 퀄리티 향상을 보장할 수 있을까? 무언가 부족하다. 

### 부족한 부분

- 장애의 근본 원인이 잘못된 소스인가?
- 잘못된 소스를 리뷰하는 구체적인 방법은 무엇인가?
- 테스트를 강화하는 구체적인 방법은 무엇인가?

위 보고서에서 좀더 향상된 방법은 무엇일까? 

이제부터 몇가지 방법들을 살펴 보면서 위 보고서에서 어떤 부분을 보충하면 좋을지 고민해보자. 

그리고 장애뿐만 아니라 제품을 개선할 수 있는 다양한 포인트를 위한 방법이 무엇인지에 대한 도구들을 알아보자. 

## Poka yoke (실수 피하기)

### 정의 

포카요케는 일본어로 토요타의 시게오신고에 의해서 처음 고안된 것으로 '실수를 피하다' 라는 의미이다. 
이는 품질 관리 측면에서 실수를 방지하도록 행동을 제한하거나, 정확한 동작을 수행하게끔 강제하는 여러가지 제약조건을 두어 실패를 방지하는 방법을 말한다. 

이를테면 자동차 기어가 P에 있어야 시동이 걸리도록 해서 급발진이나, 안전 문제가 발생하지 않도록 제한하는 방법이 그 예라고 할 수 있다. 


### 장점

### 적용방법 

### 적용사례

### 시사점 

## Postmorterm

### 정의 

### 장점

### 적용방법

### 적용사례 

### 시사점

## 5Why

### 정의 

### 장점

### 적용방법 

### 적용사례

### 시사점

## 새로운 접근법 도입

### 목적을 명확히 

### 선진 방법의 모방

### 실수로 부터 배움이 자산이 되도록

### 숙제

## 결론 

## KTPA 
- Keep (지속성) - 잘되어 가고 있는 일은 계속 해야함 
- Problem (문제점) - 만족 스럽지 못한 상황 혹은 개선해야하는 일 
- Try (시도) - 지속하고 싶은 일과 문제점에 대한 개선책 (지속되는 일이라도 한층 더 낳은 방법을 검토)
- Action (개선책) - 시도를 받아들여 구체적으로 실천하는 일 

## 참고자료: 

- 포카요케 6가지 테크닉: https://www.cmc-consultants.com/blog/6-poka-yoke-mistake-proofing-techniques-that-most-factories-overlook
- What Is the Poka-Yoke Technique?: https://kanbanize.com/lean-management/improvement/what-is-poka-yoke
- How to Implement Poka-Yoke (Mistake-Proofing) Effectively: https://www.cmc-consultants.com/blog/how-to-implement-poka-yoke-mistake-proofing-effectively
- The Ultimate Guide to Poka Yoke: https://tulip.co/resources/poka-yoke/

- postmorterm: https://brunch.co.kr/@svillustrated/13
- 전문가 기고: 실리콘밸리 기업의 실패 대응법: https://news.kotra.or.kr/user/globalBbs/kotranews/8/globalBbsDataView.do?setIdx=246&dataIdx=163218
- 에어비엔비 사례: https://news.sbs.co.kr/news/endPage.do?news_id=N1005164080
- 커밋실수사례: https://medium.com/kimjimin-company/%ED%8F%AC%EC%8A%A4%ED%8A%B8%EB%AA%A8%ED%85%9C-1-%EC%BB%A4%EB%B0%8B-%EC%8B%A4%EC%88%98%EB%A1%9C-%EC%9D%B8%ED%95%9C-%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%9E%A5%EC%95%A0-f60e778680a6

- 5why: https://m.blog.naver.com/toto_0093/220796522798
- 5why: https://m.blog.naver.com/jiwoo6941/220246908212
- 5why 기법: https://thod.tistory.com/entry/5why-design-research-techniques
- 브런치 5why: https://brunch.co.kr/@hfeel/19
- 도요타 5why: https://ibetfuture.tistory.com/96
- [자본시장 속으로] 기업의 양성평등과 5Why: https://www.etoday.co.kr/news/view/1944050
- how to use 5why: https://www.mindtools.com/pages/article/newTMC_5W.htm
- DETERMINE THE ROOT CAUSE: 5 WHYS : https://www.isixsigma.com/tools-templates/cause-effect/determine-root-cause-5-whys/
- The 5 Whys Process We Use to Understand the Root of Any Problem: https://buffer.com/resources/5-whys-process/
- How to conduct a 5 why: https://blog.thinkreliability.com/how-to-conduct-a-5-why
- What is the 5 Whys Analysis?: https://blog.infraspeak.com/5-whys-analysis/
- 5-Why Examples [The Best and The Worst!] : https://www.taproot.com/best-5-why-examples/






