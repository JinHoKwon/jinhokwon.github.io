---
title: Layered architecture
comments: true
tags: [DDD, architecture]
categories: [DDD, architecture]
header:
  teaser: "/assets/images/architecture.png"
---

본 문서에서는 Layered Architecture에 대해서 소개하고 있습니다.

<br/>

## 1. Layered Architecture



Layered Architecture는

**`구성요소들의 역활을 고려하여 레이어 단위로 분리`**함으로서, 

기능 구현시 다른 사람이 코드를 읽고 이해하기 쉽게 만들기 위해서 도입된 Architecture 이며, 

관례적으로, 네 가지 개념적 개층이 존재합니다. 

<br/>

![layered_architecture](/assets/images/architecture/layered_architecture.png)

<br/>

### 1-1. 계층 간 역활 

* User Interface : 사용자의 요청을 하위레이어로 전달, @Controller (예 : HomeController, HttpAuthController 등)
* Application : 비지니스 로직을 처리, @Service (예 : HomeService, HttpAuthService 등)
* Domain : 도메인, @Entity (예 : DTO, VO, Entity 등)
* Infrastructure : 외부 서비스, @Component (예 : Kafka, Redis, Elasticsearch 등)

<br/>

<br/>

### 1-2. 계층 간 관계 

각 구성요소별 의존성은 오직 한 방향으로만 둬서 느슨하게 결합되는 구조이며,

상위 계층은 하위 계층의 의존성을 갖을 수 있지만, (예 : HomeService 에서 Redis 호출하는 경우는 가능함.)

하위 계층은 가급적 상위 계층의 의존성을 갖으면 안됩니다. (예 : Redis 에서 HomeService 호출하는 경우는 안됨.)

<br/>

부득이하게 하위 수준의 객체가 상위 수준의 객체와 소통해야 할 경우에는 콜백, 리스너 등을 활용하는 방법을 

고려해야 합니다.

