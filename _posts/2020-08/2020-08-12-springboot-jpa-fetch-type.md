---
title: SpringBoot JPA FetchType 그리고 N+1 문제점 발견
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

본 문서에서는 JPA Entity Fetch 방식과 주의사항에 대해서 소개하고 있습니다. <br/>

## 1. JPA FetchType 사전 준비

본 섹션에서는 설명의 편의상 Parent 와 Child 엔티티를 기준으로 설명을 진행하고 있습니다.

```java
@Entity
@Table(name="parent")
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PUBLIC)
@RequiredArgsConstructor
public class Parent {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "parent_id")
    long id;
    
    @Column(name = "name", unique = true, length = 64)
    @NonNull
    String name;
    
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.EAGER)
    @NonNull
    Set<Child> children = new LinkedHashSet<>();
}

@Entity
@Table(name = "child")
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PUBLIC)
@ToString(of = {"id", "name"})
@RequiredArgsConstructor
public class Child {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name="child_id")
    long id;

    @Column(name="name")
    @NonNull
    String name;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @NonNull
    @JoinColumn(name = "parent_id")
    Parent parent;
}
```

그리고, DB에는 다음과 같은 데이터가 미리 입력되어 있다고 가정하겠습니다.

```sql
CREATE TABLE `parent` (
	`parent_id` BIGINT(19,0) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(64) NULL DEFAULT NULL COLLATE 'utf8_general_ci',
	PRIMARY KEY (`parent_id`) USING BTREE,
	UNIQUE INDEX `UK_tkff6d2q9uuhuhs0gya31w046` (`name`) USING BTREE
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB
AUTO_INCREMENT=7
;

CREATE TABLE `child` (
	`child_id` BIGINT(19,0) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(255) NULL DEFAULT NULL COLLATE 'utf8_general_ci',
	`parent_id` BIGINT(19,0) NULL DEFAULT NULL,
	PRIMARY KEY (`child_id`) USING BTREE,
	INDEX `FK7dag1cncltpyhoc2mbwka356h` (`parent_id`) USING BTREE,
	CONSTRAINT `FK7dag1cncltpyhoc2mbwka356h` FOREIGN KEY (`parent_id`) REFERENCES `test`.`parent` (`parent_id`) ON UPDATE NO ACTION ON DELETE NO ACTION
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB
AUTO_INCREMENT=13
;

insert into parent (parent_id, name) values (1, 'parent1');
insert into parent (parent_id, name) values (2, 'parent2');
insert into parent (parent_id, name) values (3, 'parent3');

insert into child (child_id, name, parent_id) values (1, 'child1-1', 1);
insert into child (child_id, name, parent_id) values (2, 'child1-2', 1);

insert into child (child_id, name, parent_id) values (3, 'child2-1', 2);
insert into child (child_id, name, parent_id) values (4, 'child2-2', 2);

insert into child (child_id, name, parent_id) values (5, 'child3-1', 3);
insert into child (child_id, name, parent_id) values (6, 'child3-2', 3);
```

<br/>

## 2. JPA Transaction 의 이해

`@Transactional`을 생략한 경우, 질의 실행 시점마다 새로운 트랜잭션이 시작됩니다.<br/>

예를 들어,  다음과 같이 (1), (2)가 실행된 경우

```java
@Test
public void parentLazyChildLazyWithoutTransactionalTest1() throws Exception {
    Child child1 = childRepository.findById(1L).get(); // (1)
    Child child2 = childRepository.findById(2L).get(); // (2)
}
```

DB상 로그는 다음과 같습니다. 

```sql
(1) 실행된 경우
2020-08-12T00:06:26.123292Z       218 Query     set session transaction read only
2020-08-12T00:06:26.126612Z       218 Query     SET autocommit=0
2020-08-12T00:06:26.633422Z       218 Query     select child0_.child_id as child_id1_0_0_, child0_.name as name2_0_0_, child0_.parent_id as parent_i3_0_0_ from child child0_ where child0_.child_id=1
2020-08-12T00:06:27.032045Z       218 Query     commit
2020-08-12T00:06:27.035878Z       218 Query     SET autocommit=1
2020-08-12T00:06:27.039817Z       218 Query     set session transaction read write

(2) 실행된 경우
2020-08-12T00:06:56.024440Z       218 Query     set session transaction read only
2020-08-12T00:06:56.027825Z       218 Query     SET autocommit=0
2020-08-12T00:06:56.040611Z       218 Query     select child0_.child_id as child_id1_0_0_, child0_.name as name2_0_0_, child0_.parent_id as parent_i3_0_0_ from child child0_ where child0_.child_id=2
2020-08-12T00:06:56.053236Z       218 Query     commit
2020-08-12T00:06:56.057291Z       218 Query     SET autocommit=1
2020-08-12T00:06:56.061342Z       218 Query     set session transaction read write
```

이러한 특징으로 인해서, JPA 환경에서는 이미 트랜잭션이 종료된 엔티티 객체로 부터 연관된 엔티티를 조회하게 될 경우<br/>

`LazyInitializationException - no sessions`예외가 발생하게 됩니다.<br/>

<br/>

그리고, 이와 같은 `LazyInitializationException` 을 방지하기 위해서는 <br/>

연관된 엔티티를 즉시로딩하는 FetchType.EAGER 유형의 fetch 를 사용하는 방법과 <br/>

`@Transactional`어노테이션을 사용하여, 한 트랜잭션 내에서 모두 처리하는 방법이 있습니다.<br/>

<br/>

## 3. JPA FetchType.EAGER

연관된 엔티티를 즉시로딩하는 Fetch 방식입니다. <br/>

이 때, 연관된 엔티티가 EAGER 방식으로 설정되어 있지 않는 경우, 지연로딩 방식으로 동작합니다.<br/>

<br/>

#### 3-1. Parent LAZY, Child EAGER 로 설정된 경우

Child 엔티티에서 Parent엔티티를 지연로딩하기 때문에,<br/>

getParent() 메소드를 호출하는 순간 Parent객체가 아닌,<br/>

Parent객체를 감싼 HibernateProxy 객체를 반환하게 됩니다.

```java
public class Parent {
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.EAGER)
    @NonNull
    Set<Child> children = new LinkedHashSet<>();
}

public class Child {
    @ManyToOne(fetch = FetchType.LAZY)
    @NonNull
    @JoinColumn(name = "parent_id")
    Parent parent;
}


@Test
public void parentLazyChildEagerTest() throws Exception {
    Child child = childRepository.findById(1L).get();		// (1) 
    Parent parent = child.getParent();
    Assertions.assertTrue(parent instanceof HibernateProxy);
    Assertions.assertThrows(LazyInitializationException.class, () -> {
        log.info("{}", parent.getName());					// (2)
    });
}
```

그런데, 문제는 이미 (1) 번 시점에 Session 은 종료되었기 때문에,<br/>

반환된 HibernateProxy 객체로 더 이상 진행할 수 있는 작업은 없는 상태가 되버립니다.<br/>

그런 상황에서 (2)번과 같은 작업을 요청하게 되면, `LazyInitializationException - no sessions` 예외가 발생하게 됩니다.<br/>

<br/>

#### 3-2. Parent EAGER, Child LAZY 로 설정된 경우

Child 엔티티를 조회할 때, Parent의 Entity의 FetchType이 EAGER로 설정되었기 때문에,<br/>

Parent 엔티티까지 포함하여 함께 조회 합니다. <br/>

때문에, 아래 코드는 정상적으로 동작하게 됩니다. 

```java
public class Parent {
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
    @NonNull
    Set<Child> children = new LinkedHashSet<>();
}

public class Child {
    @ManyToOne(fetch = FetchType.EAGER)
    @NonNull
    @JoinColumn(name = "parent_id")
    Parent parent;
}

@Test
public void parentEagerChildLazyTest() throws Exception {
    // 
    Child child = childRepository.findById(1L).get();
    Parent parent = child.getParent();
    Assertions.assertTrue(parent instanceof Parent);
    log.info("{}", parent.getName());
}
```

이 때, 실행되는 DB쿼리는 다음과 같습니다.

```sql
2020-08-12T00:44:29.861439Z       298 Query     set session transaction read only
2020-08-12T00:44:29.865337Z       298 Query     SET autocommit=0
2020-08-12T00:44:30.384531Z       298 Query     select child0_.child_id as child_id1_0_0_, child0_.name as name2_0_0_, child0_.parent_id as parent_i3_0_0_, parent1_.parent_id as parent_i1_1_1_, parent1_.name as name2_1_1_ from child child0_ left outer join parent parent1_ on child0_.parent_id=parent1_.parent_id where child0_.child_id=1
2020-08-12T00:44:30.744781Z       298 Query     commit
2020-08-12T00:44:30.748240Z       298 Query     SET autocommit=1
```

<br/>

#### 3-3. Parent EAGER, Child EAGER 로 설정된 경우

상호 연관된 모든 엔티티가 EAGER 방식으로 설정된 경우에 특정 엔티티를 조회할 때, <br/>

연관된 엔티티까지 함께 포함하여 조회하기 때문에 다음 코드는 정상동작하게 됩니다.

```java
public class Parent {
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.EAGER)
    @NonNull
    Set<Child> children = new LinkedHashSet<>();
}

public class Child {
    @ManyToOne(fetch = FetchType.EAGER)
    @NonNull
    @JoinColumn(name = "parent_id")
    Parent parent;
}

@Test
public void parentEagerChildEagerTest() throws Exception {
    Child child = childRepository.findById(1L).get();
    Parent parent = child.getParent();
    Assertions.assertTrue(parent instanceof Parent);
    log.info("{}", parent.getName());
}
```

이 때, 실행되는 DB쿼리는 다음과 같습니다.

```sql
2020-08-12T00:46:31.187381Z       318 Query     set session transaction read only
2020-08-12T00:46:31.190870Z       318 Query     SET autocommit=0
2020-08-12T00:46:31.616819Z       318 Query     select child0_.child_id as child_id1_0_0_, child0_.name as name2_0_0_, child0_.parent_id as parent_i3_0_0_, parent1_.parent_id as parent_i1_1_1_, parent1_.name as name2_1_1_ from child child0_ left outer join parent parent1_ on child0_.parent_id=parent1_.parent_id where child0_.child_id=1
2020-08-12T00:46:31.987566Z       318 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=1
2020-08-12T00:46:32.038648Z       318 Query     commit
```

반면에, `findById`가 아닌 `getOne`메소드를 호출한 경우에는 `LazyInitializationException - no sessions` 예외가 발생합니다.

이는 getOne메소드 같은경우, HibernateProxy를 사용하여 데이터를 지연로딩하는 방식으로 데이터를 읽어들이기 때문입니다.

```java
@Test
public void parentEagerChildEagerGetOneTest() throws Exception {
    Child child = childRepository.getOne(1L);
    Assertions.assertTrue(child instanceof HibernateProxy);

    Assertions.assertThrows(LazyInitializationException.class, () -> {
        Parent parent = child.getParent();
    });
}
```

정리하면, 상호연관된 객체의 FetchType이 EAGER로 설정된 경우라도, 지연로딩하고 싶으면 `getOne`메소드를 호출하면 됩니다.<br/>

다만 해당 시점에 Session이 유지되고 있어야 합니다.<br/>

<br/>

#### 3-4. Parent EAGER, Child EAGER 로 설정된 경우, Collection을 로딩하는 경우

이번에는 getOne과 같이 1개가 아닌, 모든 엔티티를 조회해 보겠습니다.<br/>

실행결과 오류는 없었고, 정상 수행되었습니다.

```java
public class Parent {
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.EAGER)
    @NonNull
    Set<Child> children = new LinkedHashSet<>();
}

public class Child {
    @ManyToOne(fetch = FetchType.EAGER)
    @NonNull
    @JoinColumn(name = "parent_id")
    Parent parent;
}

@Test
public void parentEagerChildEagerCollectionTest() throws Exception {
    List<Child> children = childRepository.findAll();
    Child child = children.get(0);
    Parent parent = child.getParent();
    log.info("parent name : {}", parent.getName());
}
```

하지만, 실행된 이후, DB 쿼리 수행내역을 살펴보면, <br/>

N개의 엔티티를 조회하기 위해서 N+1 쿼리를 수행하였음을 확인할 수 있습니다.

```sql
2020-08-12T00:50:17.352501Z       348 Query     set session transaction read only
2020-08-12T00:50:17.353372Z       348 Query     SET autocommit=0
2020-08-12T00:50:17.401834Z       348 Query     select child0_.child_id as child_id1_0_, child0_.name as name2_0_, child0_.parent_id as parent_i3_0_ from child child0_
2020-08-12T00:50:17.435176Z       348 Query     select parent0_.parent_id as parent_i1_1_0_, parent0_.name as name2_1_0_, children1_.parent_id as parent_i3_0_1_, children1_.child_id as child_id1_0_1_, children1_.child_id as child_id1_0_2_, children1_.name as name2_0_2_, children1_.parent_id as parent_i3_0_2_ from parent parent0_ left outer join child children1_ on parent0_.parent_id=children1_.parent_id where parent0_.parent_id=1
2020-08-12T00:50:17.459956Z       348 Query     select parent0_.parent_id as parent_i1_1_0_, parent0_.name as name2_1_0_, children1_.parent_id as parent_i3_0_1_, children1_.child_id as child_id1_0_1_, children1_.child_id as child_id1_0_2_, children1_.name as name2_0_2_, children1_.parent_id as parent_i3_0_2_ from parent parent0_ left outer join child children1_ on parent0_.parent_id=children1_.parent_id where parent0_.parent_id=2
2020-08-12T00:50:17.463340Z       348 Query     select parent0_.parent_id as parent_i1_1_0_, parent0_.name as name2_1_0_, children1_.parent_id as parent_i3_0_1_, children1_.child_id as child_id1_0_1_, children1_.child_id as child_id1_0_2_, children1_.name as name2_0_2_, children1_.parent_id as parent_i3_0_2_ from parent parent0_ left outer join child children1_ on parent0_.parent_id=children1_.parent_id where parent0_.parent_id=3
2020-08-12T00:50:17.469293Z       348 Query     commit
2020-08-12T00:50:17.470112Z       348 Query     SET autocommit=1
```

이는 JPA 환경에서 `Collection을 로딩하는 경우에는 지연로딩`이 적용되기 때문에 발생하는 현상입니다.<br/>

<br/>

## 4. JPA FetchType.LAZY

연관된 엔티티를 지연로딩하는 Fetch 방식이며, <br/>

지연로딩시점에 Session이 없는 경우, `LazyInitializationException - no sessions` 에러가 발생합니다.<br/>

<br/>

#### 4-1. Parent LAZY, Child LAZY 로 설정된 경우

`@Transactional`로 묶여있지 않는 상황이기 때문에, <br/>

(1)시점에 데이터를 읽어들이고, 트랜잭션은 종료하게 됩니다.<br/>

```java
public class Parent {
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
    @NonNull
    Set<Child> children = new LinkedHashSet<>();
}

public class Child {
    @ManyToOne(fetch = FetchType.LAZY)
    @NonNull
    @JoinColumn(name = "parent_id")
    Parent parent;
}

@Test
public void parentLazyChildLazyWithoutTransactionalTest2() throws Exception {
    Child child = childRepository.findById(1L).get();	// (1)
    Parent parent = child.getParent();
    Assertions.assertTrue(parent instanceof HibernateProxy);
    Assertions.assertThrows(LazyInitializationException.class, () -> {
        String name = parent.getName();					// (2) 
    });
}
```

즉, (1)번 이하의 실행은 Session이 없는 상황이며,<br/>

(2) 시점에는 parent 변수와 매핑된 Session이 없기 때문에 `LazyInitializationException`이 발생하게 됩니다.<br/>

(1) 시점에 실행된 DB 쿼리는 다음과 같습니다.

```sql
2020-08-12T01:48:53.573971Z       408 Query     set session transaction read only
2020-08-12T01:48:53.577652Z       408 Query     SET autocommit=0
2020-08-12T01:48:54.019908Z       408 Query     select child0_.child_id as child_id1_0_0_, child0_.name as name2_0_0_, child0_.parent_id as parent_i3_0_0_ from child child0_ where child0_.child_id=1
2020-08-12T01:48:54.409260Z       408 Query     commit
2020-08-12T01:48:54.412468Z       408 Query     SET autocommit=1
```

반면에, 다음과 같이 `@Transactional`로 처리된 경우에는 정상적으로 동작하게 됩니다.

```java
    @Test
    @Transactional
    public void parentLazyChildLazyWithTransactionalTest1() throws Exception {
        Child child = childRepository.findById(1L).get();		// (1)
        Parent parent = child.getParent();						// (2)
        Assertions.assertTrue(parent instanceof Parent);
        String name = parent.getName();
        log.info("parent name : {}", name);
    }
```

(1) 번 시점에 데이터를 읽어들이고, 트랜잭션이 유지가 됩니다. <br/>

그리고, (1) 번 시점에서 수행된 쿼리는 다음과 같습니다.

```sql
2020-08-12T00:33:40.867538Z       248 Query     select child0_.child_id as child_id1_0_0_, child0_.name as name2_0_0_, child0_.parent_id as parent_i3_0_0_ from child child0_ where child0_.child_id=1
```

<br/>

(2) 번 과정에서 수행된 쿼리

```sql
2020-08-12T00:33:59.761449Z       248 Query     select parent0_.parent_id as parent_i1_1_0_, parent0_.name as name2_1_0_ from parent parent0_ where parent0_.parent_id=1
2020-08-12T00:33:59.761449Z       248 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=1
```

하지만, 지연로딩의 특성상 N+1 문제가 다시 한 번 발생하게 됩니다.

<br/>



## 5. 정리

FetchType과 무관하게 `getOne` 호출 및 `Collection` 조회시에는 지연로딩 현상이 발생합니다.<br/>

그리고 지연로딩 시점에 Session이 유지되지 않는 경우 `LazyInitializationException - no sessions`예외가 발생합니다.<br/>

지연로딩 문제를 해결하기 위해서 `@Transactional`로 묶을 경우, N+1 현상이 발생하게 되며, 필요시 그 에 따른 대응이 필요하게 됩니다.<br/>

<br/>

다음 시간에는 N+1 문제를 해결하기 위한 방법에 대해서 설명하겠습니다.





