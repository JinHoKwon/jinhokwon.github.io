---
title: SpringBoot ORM, JPA, Hibernate
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

본 문서에서는 객체와 DB간의 관계를 매핑시켜주는 ORM(Object-relational mapping) 에 대해서 소개하고 있습니다.

## 1. ORM(Object-relational mapping)

객체와 DB 간의 관계를 매핑시켜 주는 기술로,<br/>
생산성 및 유지보수가 수월하기 때문에 업계에서 많이 사용되는 표준 기술 스펙중의 하나입니다.<br/>

<br/>

![orm](/assets/images/springboot/jpa/orm.png)

<br/>

## 2. ORM 장점과 단점

#### 2-1. ORM 장점

- 객체 지향적인 코드로 직관적인 코드 작성이 가능
- 특정 DB에 대한 의존성이 줄어듬.
- 특정 객체의 CRUD와 같은 반복적인 SQL 작업을 ORM이 대신 진행해줌.
- SQL 쿼리에 익숙하지 않은 경우, 사용하기에 적합함.

<br/>

#### 2-2. ORM 단점

- 기존 SQL쿼리에 익숙할수록, ORM에 적응하는 과정이 불편하게 느낄수 있음.
- 객체간의 연결관계가 복잡할 경우, ORM에 적용하는 과정도 점점 더 난해함.
- 객체간의 연결관계를 주의 깊에 설계하지 않은 경우, 성능이 저하됨.
- DB 중심적으로 설계되어있거나, 프로시저가 많은 환경에는 ORM을 적용하기가 쉽지 않음. 
- 리포트 및 통계와 같은 대형 쿼리를 수행할 경우, ORM 만으로는 성능 튜닝의 한계가 있음. 
  이 경우, 배치성 쿼리는 ORM 대신, SQL 쿼리 구문을 직접 튜닝 및 실행.
- 최적화를 하면 할 수록 ORM의 한계를 느끼게 됨.

<br/>

## 3. JPA (Java Persistence API)

Java 기반의 ORM 기술 표준 인터페이스이며, 해당 JPA를 구현한 기술이 Hibernate 입니다. 

즉, ORM > JPA > Hibernate

<br/>

<br/>

## 4. 정리

JPA는 기존 SQL 중심의 개발과정을 좀 더 객체 중심적으로 접근하기 위해서 소개된 기술이기는 하지만,

JPA가 모든 문제를 해결해주지는 않습니다.  <br/>

<br/>

JPA를 도입하더라도, 여전히 SQL 최적화에 대한 고민은 계속 필요하고,<br/>

필요에 따라서는 SQL 쿼리를 JPA와 병행해서 함께 사용해야 하는 경우도 있기 때문입니다.<br/>

<br/>

또한,  JPA를 소개하는 글들을 찾아보면, JPA의 수 많은 옵션을 생략한체<br/>

JPA의 편리함만을 강조하는 글들이 참 많은데요. <br/>

<br/>

그런 글을 접할때는 항상 `왜(WHY) 이렇게 되는거지?`라는 질문을 던지셔야 합니다.<br/>

왜냐하면, 편리함속에 감쳐진 옵션중 일부로 운영환경에서 장애로 연결될수도 있기 때문입니다.<br/>

<br/>







