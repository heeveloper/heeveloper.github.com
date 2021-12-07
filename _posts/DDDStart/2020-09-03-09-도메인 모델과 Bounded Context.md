---
layout: post
title:  "[DDD Start]9장. 도메인 모델과 Bounded Context"
date:   2020-09-02 22:13:1
author: Heeveloper
categories: DDD
tags: DDD DomainDrivenDesign 도메인주도설계 Domain BoundedContext
excerpt: DDD Start!
mathjax : false

---

* content
{:toc}

{% include adsense_content.html %}

## 1. 도메인 모델과 경계

모델은 특정한 Context(문맥)하에서 완전한 의미를 갖는다.  
같은 제품이라도 `카탈로그 컨텍스트`와 `재고 컨텍스트`에서 의미가 서로 다르다.

<img src = "/img/DDDStart/9-domain-model-boundary.png">

이렇게 구분되는 경계를 갖는 컨텍스트를 DDD에서는 **Bounded Context**라고 부른다.

## 2. Bounded Context

Bounded Context는 모델의 경계를 결정하며, 한 개의 Bounded Context는 논리적으로 **한 개의 모델**을 갖는다.  
Bounded Context는 **용어**를 기준으로 구분한다.

이상적으로 하위 도메인과 Bounded Context가 일대일 관계를 가지면 좋겠지만 일반적으론 팀 조직 구조에 따라 결정된다.  
예를 들어, 주문 하위 도메인이라도 주문을 처리하는 팀과 복잡한 결제 금액 계산 로직을 구현하는 팀이 따로 있다고 해보자.  
이 경우 주문 하위 도메인에 `주문 Bounded Context`와 `결제 Bounded Context`가 존재하게 된다.  

<img src = "/img/DDDStart/9-multiple-bounded-context.png">

반면 한 개 팀에서 구현할 때도 있다. 이 경우엔 회원, 카탈로그, 재고 등의 여러 하위 도메인을 한 개의 Bounded Context에서 구현하게 된다.  

여러 하위 도메인을 하나의 Bounded Context에서 개발할 때 주의할 점은 하위 도메인의 모델이 뒤섞이지 않도록 하는 것이다.
이는 하위 도메인마다 구분되는 **패키지**를 갖도록 구현하여 방지할 수 있다.

<img src = "/img/DDDStart/9-sub-domain-bounded-context.png">

## 3. Bounded Context의 구현

Bounded Context은 도메인 모델뿐만 포함하는 것뿐만 아니라, 표현 영역, 응용 서비스, 인프라 영역 등을 모두 포함한다.  
도메인 모델의 구조가 바뀌면 DB테이블 스키마도 함께 변경해야 하므로 해당 테이블도 Bounded Context에 포함된다.

모든 Bounded Context를 반드시 도메인 주도(DDD)로 개발할 필요는 없다. 복잡한 도메인 로직을 갖지 않는 서비스의 경우 **CRUD 방식**으로 구현해도 된다.
즉, DAO와 데이터 중심의 밸류 객체를 이용해 리뷰 기능을 구현해도 기능을 유지보수하는 데 큰 문제가 없다.

<img src = "/img/DDDStart/9-DDD-and-CRUD.png">

{% include adsense_content.html %}

한 Bounded Context에서 두 방식을 혼합해서 사용할 수도 있다. 대표적인 예가 **CQRS 패턴**이다.  
**CQRS(Command Query Responsibility Segregation)**는 상태를 변경하는 명령 기능과 내용을 조회하는 쿼리 기능을 위한 모델을 구분하는 패턴이다.  

<img src = "/img/DDDStart/9-CQRS.png">
CQRS에 대한 내용은 11장에서 다시 살펴보도록 하자.  

각 Bounded Context는 서로 다른 구현 기술을 사용할 수도 있다.  
웹 MVC는 스프링 MVC를 사용하고 리포지터리 구현 기술로 JPA/하이버네이트를 사용하는 Bounded Context가 존재하고, Netty를 이용해서 REST API를 제공하고 MyBatis를 리포지터리 기술로 사용하는 Bounded Context가 존재할 수도 있다.

Bounded Context가 반드시 사용자에게 보여지는 UI를 갖지 않고 JSON으로 응답할 수도 있다.  
또는 별도 UI를 처리하는 서버를 두고 UI 서버에서 각 Bounded Context와 통신해서 사용자 요청을 처리하는 방법도 있다.(**파사드Facade 역할**)

## 4. Bounded Context간 통합

온라인 쇼핑 사이트에서 매출 증대를 위해 카탈로그 하위 도메인에 추천 기능을 도입했다고 하자. 기존 카탈로그 시스템을 개발하던 팀과 별도로 추천 시스템을 담당하는 팀이 생겨 만들기로 했다. 이렇게 되면 카탈로그 하위 도메인에는 기존 `카탈로그를 위한 Bounded Context`와 `추천 기능을 위한 Bounded Context`가 생기고 **Bounded Context 통합**이 발생한다.

<img src = "/img/DDDStart/9-catalog-and-recommend-context.png">

카탈로그 시스템은 추천 시스템으로부터 추천 데이터를 받아오지만, 카탈로그 시스템에선 추천 도메인 모델이 아닌 카탈로그 도메인 모델을 사용해 추천 상품을 표현해야 한다.
~~~java
// 상품 추천 기능을 표현하는 도메인 서비스
public interface ProductRecommendationService {
  public List<Product> getRecommendationsOf(ProductId id);
}
~~~

도메인 서비스를 구현한 클래스는 인프라스트럭처 영역에 위치하여, **외부 시스템과의 연동**을 처리하고 **도메인 모델간 변환**을 책임진다.

<img src = "/img/DDDStart/9-acl.png">

`RecSystemClient` 는 외부 추천 시스템이 제공하는 REST API를 이용하여 특정 상품을 위한 추천 상품 목록을 로딩하고 카탈로그 도메인에 맞는 상품 모델로 변환한다.

{% include adsense_content.html %}

REST API를 이용한 두 Bounded Context의 직접 통합 방식 외에 간접적으로 통합하는 방법도 있는데, **메세지 큐**를 이용하는 것이다. 이 때 큐에 담기는 메세지는 보통 해당 큐를 개발하는 팀의 도메인을 따르는 데이터를 기준으로 개발된다.

<img src = "/img/DDDStart/9-pub-and-sub.png">

> **마이크로서비스와 Bounded Context**
>
> 넷플릭스 아마존 등 많은 기업이 점차 마이크로서비스 아키텍처를 수용하는 추세이다. 마이크로 서비스는 애플리케이션을 작은 서비스로 나누어 개발하는 아키텍처 스타일이다.  
> 개별 서비스를 독립된 프로세스로 실행하고 각 서비스가 REST API나 메시징을 이용해서 통신하는 구조를 갖는다.  
>
> 이런 마이크로서비스의 특징은 Bounded Context와 잘 어울린다. 각 Bounded Context는 모델의 경계를 형성하는데, Bounded Context를 마이크로서비스로 구현하면 자연스럽게 컨텍스트별로 모델이 분리된다. 코드로 치면 마이크로서비스마다 프로젝트를 생성하므로 Bounded Context마다 프로젝트를 만들게 된다. 이는 코드 수준에서 모델을 분리해 두 Bounded Context의 모델이 섞이지 않게 해준다.  
>
> 별도 프로세스로 개발한 Bounded Context는 독립적으로 배포하고 모니터링하고 확장하게 되는데 이 역시 마이크로서비스의 특징이다.



## 5. Bounded Context간 관계

Bounded Context는 어떤 식으로든 연결되기 때문에 두 Bounded Context는 다양한 방식으로 관계를 맺는다.

가장 흔한 관계는 한쪽에서 API를 제공하고 한 쪽에서 그 API를 호출하는 관계이다. REST API가 대표적이다.  
이 관계에서 API를 사용하는 Bounded Context(하류downstream 컴포넌트)는 API를 제공하는 Bounded Context(상류upstream 컴포넌트)에 의존하게 된다.

상류 팀의 고객인 하류 팀이 다수 존재하면 상류 팀은 여러 하류 팀의 요구사항을 수용할 수 있는 API를 만들고 이를 서비스 형태로 공개해서 서비스의 일관성을 유지할 수 있다. 이런 서비스를 가리켜 **공개 호스트 서비스(Open Host Service, OHS)** 라고 한다.

OHS의 대표적인 예가 **검색**이다. 블로그, 카페, 게시판과 같은 서비스를 제공하는 포탈은 각 서비스별로 검색 기능을 구현하기보다는 검색을 위한 전용 시스템을 구축하고 검색 시스템과 각 서비스를 통합한다. 이 때 검색 시스템은 상류 컴포넌트가 되고 각 서비스들은 하류 컴포넌트가 된다.

<img src = "/img/DDDStart/9-ohs-search.png">

하류 컴포넌트는 상류 서비스의 모델이 자신의 도메인 모델에 영향을 주지 않도록 보호해 주는 완충 지대를 만들어야 하는데, 이는 이미 앞서 언급한 모델 변환을 해주어 내 모델이 깨지는 것을 막아 주는 **안티코럽션 계층(Anticorruption Layer, ACL)** 이 된다.

<img src = "/img/DDDStart/9-acl.png">

{% include adsense_content.html %}

두 Bounded Context가 같은 모델을 공유하는 경우도 있다. 예를 들어, 운영자를 위한 주문 관리 도구를 개발하는 팀과 고객을 위한 주문 서비스를 개발하는 팀이 다르다고 가정했을 때, 이 경우 두 팀은 `주문` 을 표현하는 모델을 공유함으로써 중복 개발을 막을 수 있다. 이렇게 공유하는 모델을 **공유 커널(Shared Kernel)** 이라고 부른다. 

공유 커널의 장점은 중복을 줄여준다는 것이다. 때문에 한 팀이 임의로 모델을 변경하거나 밀접한 관계를 형성할 수 없다면 오히려 개발이 지연되고 정체되는 문제가 더 커지게 된다. 

마지막으로 살펴볼 관계는 **독립 방식(Separate Way)** 관계이다. 그냥 서로 통합하지 않는 방식으로서 서로 독립적으로 모델을 발전시킨다. 독립 방식에서 두 Bounded Context간 통합은 **수동**으로 이뤄지는데 예를 들어, 온라인 쇼핑몰 솔루션과 외부의 ERP 서비스를 사용한다고 가정했을 때, 운영자는 쇼핑몰에서의 판매정보를 직접 ERP에 수동 입력해야 한다.  규모가 커지는 경우 중간에 통합 시스템을 별도로 구축하여 처리할 수도 있다.



## 6. 컨텍스트 맵

컨텍스트 맵은 Bounded Context 간의 관계를 표시한 것이다.  
Bounded Context 영역에 주요 애그리거트를 함께 표시하면 모델에 대한 관계가 더 명확히 드러난다. 추가로 `OHS`와 `ACL`까지 표시한다면 도메인간 조직 구조를 알 수 있어 전체 관계를 이해하는 데 도움이 된다.

컨텍스트 맵을 그리는 규칙은 따로 없고 단순하기 때문에 화이트보드나 파워포인트와 같은 도구를 이용해 쉽게 그릴 수 있다.

<img src = "/img/DDDStart/9-context-map.png">
