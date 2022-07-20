---
layout: post
title:  "@Transactional 원리"
date:   2022-07-19 23:10:00
author: Heeveloper
categories: Spring
tags: Spring Springboot Transaction Transactional
excerpt: Spring
mathjax : false
---

{% include adsense_content.html %}

* content
{:toc}

## 1. @Transactional
트랜잭션 처리가 필요한 비즈니스 로직이 있는데, 이를 매번 직접 구현을 하려고 하면 코드 중복뿐만 아니라 핵심 로직으로부터 집중이 분산될 수 있다.  
이러한 이유로 Spring에서는 `@Transactional` Annotation으로 트랜잭션 처리 기능을 제공한다.  
해당 Annotation이 적용된 클래스 또는 메서드에 대해선 내부적으로 **AOP(Aspect Oriented Programming)**를 통해 트랜잭션 처리가 적용된다.
<br>

## 2. AOP (Aspect Oriented Programming)
**Aspect**란 핵심 기능에 부가되는 특별한 모듈이다.  
**AOP**란 Application을 설계할 때 핵심 기능의 코드에 공통된 부가기능 코드를 독립적으로 분리시키는 기술을 의미힌다.  
(ex. 로깅 기능)  
- **Advice** : 부가기능 (아주 단순한 형태의 Aspect)
- **Point-cut** : 부가기능 부여대상

이러한 AOP의 방식은 크게 2가지가 있다.

### 2-1. JDK Dynamic Proxy
<img src="/img/Spring/Transactional/Proxy-pattern.svg.png">
<br>
트랜잭션 처리를 본래 클래스의 객체가 아닌 **Dynamic Proxy 객체**에 위임하는 방식이다.  
이 때 Dynamic Proxy 객체는 타겟이 상속하는 Interface를 상속 후 추상메서드를 구현하여 내부적으로 타겟 메서드 호출 전/후에 트랜잭션 처리를 수행한다.  
즉, Client(혹은 Controller)는 실제로는 프록시 객체를 호출하는 것이다.  


> **참고)**  
> AOP는 **Proxy Pattern**으로서 타겟 메서드를 컨트롤하고, 부가기능을 **Decorate Pattern**을 통해 적용한다.

프록시 객체 앞에 Dynamic이 붙는 이유는 이러한 기능을 가지는 프록시 객체가 런타임에 동적으로 생성되기 때문이며, 만일 타겟이 상속한 인터페이스가 없는 경우라면 CGLib 방식으로 해결할 수 있다.


### 2-2. CGLib
JDK Dynamic Proxy와 같은 방식은 Java Refelection을 사용하는데, CGLib은 이와 달리 바이스 생성 프레임워크를 사용해 런타임시 프록시 객체를 생성한다.  
즉, 인터페이스 대신 타겟 오브젝트를 상속하는 프록시 객체를 생성한다.  


## 3. SpringBoot 적용
application 파일을 보면 기본적으로 `proxy-target` 값이 true로 설정되어 있는데 이는 GCLib 방식을 의미한다.  
즉 인터페이스 유무에 상관없이 프록시 객체를 생성하는 것을 의미하며, 만일 JDK Dynamic Proxy 방식을 적용하고 싶다면 `proxy-target`값을 false로 설정하면 된다.

> **참고)**  
> 1. 핵심 기능을 가진 타겟 클래스가 Bean으로 생성  
> 2. 후처리기에서 Bean마다 포인트컷으로 판단  
> 3. Dynamic Proxy 객체 생성  


## 4. 요약
1. 기본적으로 트랜잭션 기능의 Advice는 이미 등록된 상태이다.
2. `@Transactional`을 명시하면 Point-cut으로 등록된다.
3. Advice와 Point-cut을 가지는 Advisor는 Bean으로 등록된다.
4. Bean 후처리기는 위 Bean을 프록시 객체로 치환한다.
5. Client는 실제로 프록시 객체를 주입받고, 프록시 개체의 메서드를 호출한다.  
<br>
<img src="/img/Spring/Transactional/Proxy.png">


> **중요**  
> 따라서 `@Transactional`은 프록시 객체의 메서드를 호출하는 것이므로 객체의 method에 들어갈 때와 나올 때 적용된다.  
> 즉, 본래 타겟 클래스에서 작업시 `@Transactional` 선언된 메서드/클래스는 내부적으로 사용될 수 없다.  
> - private method에 적용 안됨
> - 내부 호출 method에 적용 안됨
