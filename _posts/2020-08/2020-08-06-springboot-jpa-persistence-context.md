---
title: SpringBoot JPA Persistence Context
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

JPA 환경에서 Entity 를 추가하거나 수정하는 경우, <br/>

Persistence Context 내에 반영이 이루어지며,<br/>

이후 명시적으로 flush를 하거나, commit이 된 경우에는 DB에 반영이 됩니다.<br/>

<br/>

본 문서에서는 App과 DB 사이의 논리적인 버퍼와 같은 역활을 수행하는<br/>

`Persistence Context` 에 대해서 소개하고 있습니다.

<br/>

## 1. JPA Persistence Context



#### 1-1. Persistence Context의 생명주기

* 비영속성 : 영속성 컨텍스트와 전혀 관련이 없는 상태
* 영속 : 영속성 컨텍스트에 저장된 상태 (1차 캐쉬영역에 Entity가 영속화 된 상태라고도 함.)
* 준영속 : 영속성 컨텍스트에서 분리된 상태
* 삭제 : 삭제된 상태



이를 코드로 구현해보면 다음과 같습니다.

```java
Box box = new Box("red");		// 비영속성
entityManager.persist(box);		// 영속
entityManager.detach(box);		// 준영속
entityManager.remove(box);		// 삭제
```



<br/>

#### 1-2. Persistence Context 장점

* 1차 캐쉬 : flush 하기 이전까지는 Persistence Context 레이어에서만 작업이 진행됨.
* 동일성 보장 : Persistence Context 에서 동일 객체 조회시 매번 동일 객체가 조회됨.
* 쓰기 지연 : flush 하기 이전까지 쓰기 작업을 모아둠.
* 변경 감지 : Entity의 수정이 발생하였을 때, 알아서 UPDATE 작업이 진행됨.
* 지연 로딩 : 객체가 실제로 사용되는 시점에 DB 쿼리가 실행되어 객체를 조회함.

<br/>

#### 1-3. Persistence Context 주의사항

* EntityManager 는 트랜잭션 단위로 생성해야 함.
* EntityManager는 스레드 세이프하지 않아서, 스레드간에 공유가 불가능 함.
* Transaction 수행 후에는 반드시 EntityManager 를 닫아야 (entityManager.close()) DB Connection 이 반환됨.

<br/>

## 2. Persistence Context 실습

#### 2-1. Box.java

```java
package com.example.app.model;

import lombok.*;
import javax.persistence.*;

@Entity
@Table(name="box")
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PUBLIC)
@AllArgsConstructor
@ToString
public class Box {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column
    long id;

    @Column(name = "name", unique = false, length = 64)
    @NonNull
    String name;

    @Column(name = "code", unique = true, length = 64)
    @NonNull
    String code;

    @Builder
    public Box(String name, String code) {
        this.name = name;
        this.code = code;
    }
}
```

<br/>

#### 2-2. PersistenceContextTest.java

```java
package com.example.app;

import com.example.app.model.Box;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.MethodOrderer;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestMethodOrder;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.*;

@Slf4j
@ExtendWith(SpringExtension.class)
@SpringBootTest
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class PersistenceContextTest {

    @PersistenceContext
    EntityManager entityManager;

    @PersistenceUnit
    EntityManagerFactory entityManagerFactory;

    void crudTest(EntityManager em) {
        Box redBox = new Box("red", "001");
        Box blueBox = new Box("blue", "002");
        Box greenBox = new Box("green", "003");

        em.persist(redBox);
        em.persist(blueBox);     // create
        em.persist(greenBox);    // create : 트랜잭션 커밋 시점에 반영됨.
        em.find(Box.class, 10L);
        Box box = em.find(Box.class, blueBox.getId());  // read
        box.setName("Blue");     // update : 트랜잭션 커밋시 변경 감지 후 반영됨.
        em.remove(greenBox);     // delete
        em.detach(redBox);

        Assertions.assertFalse(em.contains(redBox));
        redBox = em.merge(redBox);
        Assertions.assertTrue(em.contains(redBox));
    }

    @Test
    // JPA는 트랜잭션 기반으로 동작하기 때문에, @Transactional 이 필요함.
    @Transactional(readOnly = false)
    // 결과를 확인하기 위해서, Rollback을 disable함.
    @Rollback(false)
    public void persistenceContextCRUDTest() throws Exception {
        crudTest(this.entityManager);
    }

    @Test
    public void persistenceContextManagerFactoryTest() throws Exception {
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();

        try {
            transaction.begin();
            crudTest(entityManager);
            transaction.commit();
        } catch (Exception e) {
            transaction.rollback();
        } finally {
            entityManager.close();
        }
    }
}
```

<br/>

#### 2-3. 실행 결과

```sql
2020-08-07T06:30:01.346044Z   303 Query  SET autocommit=0
2020-08-07T06:30:01.385594Z   303 Query  insert into box (code, name) values ('001', 'red')
2020-08-07T06:30:01.394631Z   303 Query  insert into box (code, name) values ('002', 'blue')
2020-08-07T06:30:01.399164Z   303 Query  insert into box (code, name) values ('003', 'green')
2020-08-07T06:30:01.414979Z   303 Query  select box0_.id as id1_0_0_, box0_.code as code2_0_0_, box0_.name as name3_0_0_ from box box0_ where box0_.id=10
2020-08-07T06:30:01.436593Z   303 Query  select box0_.id as id1_0_0_, box0_.code as code2_0_0_, box0_.name as name3_0_0_ from box box0_ where box0_.id=1
2020-08-07T06:30:01.456004Z   303 Query  update box set code='002', name='Blue' where id=2
2020-08-07T06:30:01.466010Z   303 Query  delete from box where id=3
2020-08-07T06:30:01.469904Z   303 Query  commit
2020-08-07T06:30:01.474923Z   303 Query  SET autocommit=1
2020-08-07T06:30:01.510962Z   303 Quit
```

