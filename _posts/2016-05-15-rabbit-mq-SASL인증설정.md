---
layout: post
title: Rabbit MQ SASL 인증관련
weight: 20
excerpt: "Rabbit MQ의 SASL메카니즘에 의한 인증 기능에 대해 간단히 알아본다."
tags: [rabbitMQ, oss]
modified: 2016-05-08
comments: true
category : rabbit
icon : wrench.ico
---

SASL 인증
======================

SASL
: Simple Authentication and Security Layer
: 인증기능이 추가된 텔넷 기반 프로토콜


RabbitMQ다양하 SASL 인증 메카니즘에 대해 지원한다.


Built-in mechanism

### PLAIN

대다수의 Client에서 사용하는 기본 인증 방법


### AMQPLAIN

AMQP 0-8 Specification에 정의된 Non-Standard 방식
QPid's Python Client의 default


### RABBIT-CR-DEMO



### 서버 설정

rabbitmq.config 내에 아래 환경 설정

~~~
auth_mechanisms	SASL : ['PLAIN', 'AMQPLAIN']
~~~



### Client 설정

JAVA의 경우 *javax.security.sasl* 를 이용하여 설정¸¸ Non-Oracle인 경우나 Android JAVA의 경우 해당 패키지를 제공하지 않는 경우도 있다.

RabbitMQ에서 SaslConfig 인터페이스의 구현체를 지원하므로 이를 사용하면 된다.

~~~ java
ConnectionFactory.getSaslConfig() 
ConnectionFactory.setSaslConfig(SaslConfig)
~~~