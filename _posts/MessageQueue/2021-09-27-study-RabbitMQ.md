---
layout: post
title:  "RabbitMQ - 이론"
date:   2021-09-27 20:33:20
author: Heeveloper
categories: MessageQueue
tags: RabbitMQ AMQP messagequeue
excerpt: RabbitMQ
mathjax : false
---

* content
{:toc}

프로젝트에 알람 기능이 추가되면서 Message Queuing System의 한 종류인 RabbitMQ를 공부한 내용을 정리해 보았습니다.  


RabbitMQ는 **AMQP**를 구현한 오픈소스 메세지 브로커인데, 메세지를 많은 사용자에게 전달하거나 요청에 대한 처리 시간이 길 때 해당 요청을 다른 API에게 위임하고 빠른 응답을 할 때 주로 사용합니다. 또한 MQ를 사용하여 애플리케이션 간 결합도를 낮출 수 있는 장점도 있습니다.

> ### AMQP (Advanced Message Queuing Protocol)
>
> - 인스턴스간 데이터를 서로 교환할 때 메세지 미들웨어 브로커를 통해 진행하는데, 이 때 사용되는 메세지 프로토콜.  
> - Producer -> Broker [Exchange -- Binding --> Queue] -> Consumer
> - AMQP 자체는 프로토콜을 의미하기 때문에 이를 구현한 MQ 제품들이 여러가지가 있습니다.  
>   ex) RabbitMQ, Kafka 등

<img src="/img/MessageQueue/amqp-and-rabbitmq.png" width="700" height="370">

{% include adsense_content.html %}

## 주요 개념

RabbitMQ는 아래 AMQP의 주요 개념을 구현하였습니다.

<img src="/img/MessageQueue/rabbitmq-flow.png" width="700" height="370">

- Producer : 메세지를 생성하고 발송하는 주체. 해당 메세지는 항상 Exchange를 통해 Queue에 저장됩니다.
- Consumer : 메세지를 수신하는 주체. Consumer는 Queue에 직접 접근하여 메세지를 가져옵니다.
- Queue : Producer가 발송한 메세지들이 Consumer에 전달되기 전까지 보관되는 장소. Queue는 이름으로 구분되기 때문에 같은 이름, 다른 설정으로 Queue를 생성하려고 하면 에러가 발생합니다.
  - Name : queue 이름. `amq`로 시작하는 이름은 예약어로 사용할 수 없습니다.
  - Durability
    - `durable` : 브로커 재시작되어도 디스크에 저장되어 남아있음
    - `transient` : 브러커가 재시작되면 사라진다. 단, Queue에 저장되는 메세지는 내구성을 갖지 않는다.
  - Auto delete : 마지막 Consumer가 구독을 끝내는 경우 자동으로 삭제된다.
  - Arguments : 메세지 TTL, Max Lenght 같은 추가 기능을 설정한다.
- Exchange : Producer들에게 전달받은 메세지들을 어떤 Queue들에게 발송할지를 결정하는 개체. 라우터 개념.
  - Direct Type : Routing Key가 정확히 일치하는 Queue에게 메세지 전송 (Unicast)
  - Topic Type : Routing Key 패턴이 일치하는 Queue에 메세지 전송 (Multicast)
  - Headers Type : [key:value]로 이뤄진 headers 값을 기준으로 일치하는 Queue에 메세지 전송 (Multicast)
  - Fanout Type : 해당 Exchnage에 등록된 모든 Queue에 메세지 전송 (Broadcast)
- Binding : Exchnage로 전달된 메세지는 Exchnage type에 추가로  Binding 규칙에 따라 Queue들에게 발송.


## Exchnage 4가지 타입

### Direct

Direct Exchange는 라우팅 키를 이용하여 메세지를 라우팅하는데, 하나의 큐에 여러 개의 라우팅 키를 지정할 수 있습니다.

<img src="/img/MessageQueue/direct.png" width="420" height="170">

- error 메세지만 저장소에 기록하고, info와 warning을 포함한 모든 정보를 디스플레이에 출력할 때를 나타내는 모식도.
  (C1: 저장소, C2: 디스플레이)

<img src="/img/MessageQueue/direct-fanout.png" width="420" height="170">

- Direct Exchnage에 여러 큐에 같은 라우팅 키를 지정하여 Fanout처럼 동작하게 할 수도 있습니다.

**** RabbitMQ의 Default Exchnage(Unnamed Exchnage)는 Direct 타입이며 RabbitMQ에서 생성되는 모든 Queue가 자동으로 binding됩니다. 이 때, 각 Queue의 이름이 라우팅 키로 지정됩니다.

### Topic Exchnage

<img src="/img/MessageQueue/topic.png" width="420" height="170">

- routingKey="example.orange.rabbit"인 경우 메세지는 Q1, Q2에 전달
- routingKey="example.orange.turtle"인 경우 메세지는 Q1에 전달
- routingKey="lazy.grape.rabbit"인 경우 메세지는 Q2에 한 번만 전달 (라우팅 패턴이 여러개 일치하더라도 하나의 큐에는 메세지 한 번만 전달)

### Headers Exchnage

Headers Exchnage는 Topic Exchnage와 유사하지만 라우팅을 위해 header를 쓴다는 차이가 있다.

<img src="/img/MessageQueue/headers.png" width="600" height="220">



### Fanout Exchnage

<img src="/img/MessageQueue/fanout.png">


{% include adsense_content.html %}


## MQ 및 Message 보존

RabbitMQ 서버가 종료 후 재기동하면, 기본적으로 Queue는 모두 제거됩니다. 이를 막기 위해 `durable` 옵션을 설정해야 하며, Producer가 메세지를 발송할 떄, `PERSISTENT_TEXT_PLAIN` 옵션을 주어야 메세지가 보존됩니다.

Queue 생성 예시

~~~java
rabbitmqChannel.queueDeclare(rabbitmqQueueName, true, false, false, null); //QueueName 다음의 true가 durable option
~~~

Message 발송 예시

~~~java
rabbitmqChannel.basicPublish(exchangeName, routingKey, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
~~~

## 메세지 분배 (Round-Robin Dispatching)

RabbitMQ는 Consumer가 병렬처리를 쉽게 할 수 있도록 같은 Queue를 바라보고 있는 여러 Consumer들에게 메세지를 균등하게 분배한다.  
즉, 첫 번째 메세지는 Consumer1에게, 두 번째 메세지는 Consumer2에게 분배한다. (중복 처리하지 않도록 1명에게만 전달)  
이를 통해, 메세지를 받아 처리하는 프로그램들은 **수평 확장**이 가능하며, Producer도 같은 Queue에 메세지를 전달할 수 있음으로써, 동일하게 수평확장이 가능하다.

## Prefetch Count

하나의 Queue에 여러 Consumer가 존재할 경우, Queue는 기본적으로 Round-Robin 방식으로 메세지를 분배합니다.  

만약 2개의 Consumer 중 홀수번쨰의 메세지는 처리시간이 짧고 짝수번째 메세지는 처리 시간이 매우 긴 경우, 계속해서 하나의 Consumer만 일을 하게 되는 상황이 발생할 수 있습니다.  

이를 예방하기 위해, `prefect count` 를 1로 설정해두면, 하나의 메세지가 처리되기 전(ACK를 보내기 전)에는 새로운 메세지를 받지 않게 되므로, 작업을 분산시킬 수 있습니다.

## 그 외 RabbitMQ 기능

### Vhost

Virtual Host를 통해서 하나의 RabbitMQ 인스턴스 안에 사용하고 있는 Application을 분리할 수 있습니다.  

즉 Broker에서 운영 환경(ex. dev, prod)에 따라 Users, Exchnage, Queue 등을 각각 사용할 수 있습니다.

### Connection

물리적인 TCP Connection, HTTPS -> TLS(SSL) Connection을 사용

### Channel

Consumer 어플리케이션에서 Broker로 많은 연결을 맺는 것은 바람직하지 않습니다.  

RabbitMQ는 Channel 이라는 개념을 통해 하나의 TCP 연결을 공유해서 사용할 수 있는 기능을 제공합니다.  

하지만 멀티 스레드, 멀티 프로세스를 사용하는 작업에서는 각각 별도의 Channel을 열고 사용하는 것이 바람직합니다.



## Reference

- https://www.rabbitmq.com/getstarted.html
- https://www.rabbitmq.com/tutorials/amqp-concepts.html
- https://nesoy.github.io/articles/2019-02/RabbitMQ
- https://jonnung.dev/rabbitmq/2019/02/06/about-amqp-implementtation-of-rabbitmq/
  
  <br>