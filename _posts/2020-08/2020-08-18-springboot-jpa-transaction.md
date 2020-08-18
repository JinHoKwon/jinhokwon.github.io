---
title: SpringBoot Transaction
comments: true
tags: [springboot, dev]
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

본 문서는 Spring Transaction 사용시, 사전에 알고있어야  하는 격리레밸과 전달행위에 대해서 설명하고 있습니다.<br/>

<br/>

## 1. 격리레밸 (Isolation level)

모든 트랜잭션은 다음과 같은 격리레벨중 한 가지 값을 가지게 됩니다.<br/>

<br/>

#### 1-1. ISOLATION_DEFAULT

개별적인 PlatformTransactionManager를 위한 디폴트 격리 레벨<br/>

* MSSQL : READ COMMITTED
* MYSQL : REPEATABLE READ
* ORACLE : READ COMMITTED
* H2 : READ COMMITTED

<br/>

#### 1-2. READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE 비교

| Isolation level  | Dirty read | Non-repeatable read | Phantom read |
| :--------------- | :--------: | :-----------------: | :----------: |
| Read uncommitted |     O      |          O          |      O       |
| Read committed   |     X      |          O          |      O       |
| Repeatable read  |     X      |          X          |      O       |
| Serializable     |     X      |          X          |      X       |

<br/>



## 2. 전달행위 (Propagation)

<br/>

#### 2-1. REQUIRED 

propagation의 기본 속성으로 부모 트랜잭션 있으면 참여하고 없으면 새로운 트랜잭션을 생성합니다.

<br/>

#### 2-2. SUPPORTS

부모 트랜잭션 있으면 부모 트랜잭션에 참여하며, <br/>

부모 트랜잭션이 없으면 트랜잭션 없이 진행합니다.<br/>

경계안에서 Connection이나 하이버네이트 Session 등을 공유할 수 있습니다.<br/>

<br/>

#### 2-3. MANDATORY 

부모 트랜잭션 있으면 부모 트랜잭션에 참여하고, <br/>

부모 트랜잭션이 없으면 예외를 발생합니다.<br/>

보통, 독립적으로 트랜잭션을 진행하면 안되는 경우에 사용할 수 있습니다.<br/>

<br/>

#### 2-4. REQUIRES_NEW 

항상 새로운 트랜잭션을 시작합니다. <br/>
이 때 부모트랜잭션이 존재할 경우, 부모 트랜잭션을 잠시 보류하고 새로운 트랜잭션 진행 후, <br/>
기존 트랜잭션을 이어서 진행하게 됩니다.<br/>

<br/>

#### 2-5. NEVER  

시작된 트랜잭션 있으면 예외 발생(트랜잭션을 사용하지 않도록 강제)<br/>

<br/>



#### 2-6. NESTED

시작된 트랜잭션 있으면 중첩 트랜잭션을 시작(트랜잭션 안에 트랜잭션 생성)<br/>

부모 트랜잭션의 커밋과 롤백은 자식에게 영향을 주지만, 반대의 경우는 영향 없음<br/>

<br/>

## 3. 주의사항

#### 3-1. readonly = true 인 상태에서 insert를 하는 경우

```java
@Slf4j
@DataJpaTest
@ActiveProfiles("test")
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Transactional(propagation = NOT_SUPPORTED)
public class ReadOnlyTests {

    @Autowired
    PlatformTransactionManager transactionManager;

    @Autowired
    EntityManager entityManager;

    @Test
    void readOnlyIsTrueTest() throws Exception {
        TransactionTemplate  transactionTemplate = new TransactionTemplate(transactionManager);
        transactionTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
        transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_DEFAULT);
        transactionTemplate.setReadOnly(true);

        Assertions.assertThrows(PersistenceException.class, () -> {
            transactionTemplate.executeWithoutResult(transactionStatus ->  {
                entityManager.persist(new Box("toy"));
            });
        });
    }
}
```

<br/>

#### 3-2. Required 사용시, 서브 트랜잭션의 예외로 모두 다 롤백되는 상황

```java
void bothParentAndChildInsertSameKeyWithRequiredTransactionTest() throws Exception {
    TransactionTemplate parentTransactionTemplate = new TransactionTemplate(transactionManager,
                                                                            new SimpleTransactionDefition(PROPAGATION_REQUIRED, ISOLATION_DEFAULT));

    TransactionTemplate childTransactionTemplate = new TransactionTemplate(transactionManager,
                                                                           new SimpleTransactionDefition(PROPAGATION_REQUIRED, ISOLATION_DEFAULT));

    Assertions.assertThrows(PersistenceException.class, () -> {
        parentTransactionTemplate.executeWithoutResult(parentTransactionStatus -> {
            entityManager.persist(new Box("same name"));
            childTransactionTemplate.executeWithoutResult(childTransactionStatus -> {
                // 트랜잭션이 공유되기 때문에,
                // 한개라도 롤백시 이전것까지 포함하여 한꺼번에 롤백처리됨.
                entityManager.persist(new Box("same name"));
            });
        });
    });

    List<Box> boxList =  entityManager.createQuery("SELECT b FROM Box b", Box.class).getResultList();
    Assertions.assertEquals(0, boxList.size());
}
```

<br/>

#### 3-3. REQUIRES_NEW 사용시, 서로 다른 트랜잭션에서 동일한 테이블에 INSERT를 하는 경우

특정 테이블에 이미 트랜잭션이 시작된 상태에서, 트랜잭션 안에서 또 다시 동일한 특정 테이블의 동일한 값을 입력하는 경우,<br/>

서브 트랜잭션에서는 무한대기후, Lock wait timeout exceeded 예외가 발생하게 됩니다.<br/>

<br/>

이러한 경우를 방지하기 위해서는 PROPAGATION_REQUIRES_NEW 와 같은 Propagation 옵션 사용시에는,<br/>

반드시, 서브 트랜잭션에서 상위 트랜잭션과 공유되는 테이블은 없는지 사전에 잘 체크해 보아야 합니다.

```java
@Test
void bothParentAndChildInsertSameTableRequiresNewTransactionTest() throws Exception {
    TransactionTemplate parentTransactionTemplate = new TransactionTemplate(transactionManager,
                                                                            new SimpleTransactionDefition(PROPAGATION_REQUIRES_NEW, ISOLATION_DEFAULT));

    TransactionTemplate childTransactionTemplate = new TransactionTemplate(transactionManager,
                                                                           new SimpleTransactionDefition(PROPAGATION_REQUIRES_NEW, ISOLATION_DEFAULT));

    Assertions.assertThrows(PessimisticLockException.class, () -> {
        parentTransactionTemplate.executeWithoutResult(parentTransactionStatus -> {
            // 매 작업마다 트랜잭션이 생성됨.
            entityManager.persist(new Box("same name"));
            childTransactionTemplate.executeWithoutResult(childTransactionStatus -> {
                // 이미, 'box' 테이블에 락이 걸려있기 때문에,
                // 다음 작업은 timeout 예외가 발생하게 됩니다.
                // SQL Error: 1205, SQLState: 40001
                // Lock wait timeout exceeded; try restarting transaction
                entityManager.persist(new Box("same name"));
            });
        });
    });

    List<Box> boxList =  entityManager.createQuery("SELECT b FROM Box b", Box.class).getResultList();
    Assertions.assertEquals(0, boxList.size());
}
```



