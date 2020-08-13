---
title: SpringBoot JPA N+1 문제와 해결방법
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

본 문서에서는 JPA 환경에서 자주 접하는 N+1 문제와 해결방법에 대해서 설명하고 있습니다.<br/>

## 1. JPA N+1 문제와 해결방법 

#### 1-1. N+1 문제의 정의 

N개의 연관된 엔티티를 조회할 때, N+1 번 쿼리를 수행하는 현상을 N+1 문제라고 정의합니다. <br/>

로딩이 필요한 시점에 필요한 엔티티만 로딩하는 JPA의 지연로딩 특징으로 인하여 발생하는 현상이며,<br/>

해당 시점에 발생하는 DB 쿼리는 대략 다음과 같습니다.<br/>

```sql
2020-08-13T00:53:05.478201Z       179 Query     select parent0_.parent_id as parent_i1_1_, parent0_.name as name2_1_ from parent parent0_
2020-08-13T00:53:05.539690Z       179 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=1
2020-08-13T00:53:05.553336Z       179 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=2
2020-08-13T00:53:05.556093Z       179 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=3
```

간단하게는 즉시로딩(FetchType.EAGER)방식을 적용하면, 손쉽게 N+1 문제가 해결될 것 같지만,<br/>

Collection 조회시 지연로딩이 적용되는 JPA 특징으로 인하여, 이 또한 N+1 문제가 동일하게 발생합니다.<br/>

<br/>

즉, 연관된 엔티티를 전체조회할 때는 반드시 N+1 문제를 확인하고, 필요시 대응을 해주어야 합니다.

<br/>

#### 1-2. N+1 문제의 해결방법

N+1 문제를 해결하는 방법에는 `JOIN FETCH` 구문을 사용하는 방법과 `@EntityGraph`를 사용하는 방법이 있습니다.

```java
package com.example.app.repository;

import com.example.app.model.Parent;
import com.sun.istack.NotNull;
import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import javax.persistence.Entity;
import java.util.List;
import java.util.Set;

@Repository
public interface ParentRepository extends JpaRepository<Parent, Long> {
    Parent findByName(String name);
    List<Parent> findAll();

    @Query("SELECT parent FROM Parent parent JOIN FETCH parent.children")
    Set<Parent> findAllJoinFetch();

    @EntityGraph(attributePaths = {"children"})
    @Query("SELECT DISTINCT parent FROM Parent parent")
    List<Parent> findAllByEntityGraph();
}
```

이 둘의 가장 큰 차이는 JPA 실행 시점에 기록되는 쿼리를 보면 알 수 있습니다.

##### JOIN FETCH 구문 사용시 SQL : inner join 사용

```sql
2020-08-13T00:55:12.907239Z       199 Query     select parent0_.parent_id as parent_i1_1_0_, children1_.child_id as child_id1_0_1_, parent0_.name as name2_1_0_, children1_.name as name2_0_1_, children1_.parent_id as parent_i3_0_1_, children1_.parent_id as parent_i3_0_0__, children1_.child_id as child_id1_0_0__ 
from parent parent0_ inner join child children1_ on parent0_.parent_id=children1_.parent_id
```

##### @EntityGraph 구문 사용시 SQL : left outer join 사용

```sql
2020-08-13T00:58:44.113453Z       219 Query     select distinct parent0_.parent_id as parent_i1_1_0_, children1_.child_id as child_id1_0_1_, parent0_.name as name2_1_0_, children1_.name as name2_0_1_, children1_.parent_id as parent_i3_0_1_, children1_.parent_id as parent_i3_0_0__, children1_.child_id as child_id1_0_0__ 
from parent parent0_ left outer join child children1_ on parent0_.parent_id=children1_.parent_id
```

<br/>

## 2. JPA N+1 문제 재현을 위한 테스트 샘플

#### 2-1. Parent.java

```java
package com.example.app.model;

import lombok.*;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.LinkedHashSet;
import java.util.List;
import java.util.Set;

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

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
    @NonNull
    Set<Child> children = new LinkedHashSet<>();

    @Override
    public String toString() {
        return id + ":" + name + ":" + children.size();
    }
}
```

#### 2-2. Child.java

```java
package com.example.app.model;

import lombok.*;
import javax.persistence.*;

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

#### 2-3. ParentRepository.java

```java
package com.example.app.repository;

import com.example.app.model.Parent;
import com.sun.istack.NotNull;
import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import javax.persistence.Entity;
import java.util.List;
import java.util.Set;

@Repository
public interface ParentRepository extends JpaRepository<Parent, Long> {
    Parent findByName(String name);
    List<Parent> findAll();

    @Query("SELECT parent FROM Parent parent JOIN FETCH parent.children")
    Set<Parent> findAllJoinFetch();

    @EntityGraph(attributePaths = {"children"})
    @Query("SELECT DISTINCT parent FROM Parent parent")
    List<Parent> findAllByEntityGraph();
}
```

#### 2-4. ChildRepository.java

```java
package com.example.app.repository;

import com.example.app.model.Child;
import com.example.app.model.Parent;
import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Set;

@Repository
public interface ChildRepository extends JpaRepository<Child, Long> {
    Child findByName(String name);
}
```

#### 2-5. SpringBootConsoleApplicationTests.java

```java
package com.example.app;

import com.example.app.model.Child;
import com.example.app.model.Parent;
import com.example.app.repository.ChildRepository;
import com.example.app.repository.ParentRepository;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.transaction.annotation.Transactional;

@Slf4j
@ExtendWith(SpringExtension.class)
@SpringBootTest
public class SpringBootConsoleApplicationTests {

    @Test
    @Transactional
    @Rollback(false)
    public void parentEagerChildEagerCollectionTest() throws Exception {
        /*
        즉시 로딩 : N+1 문제가 발생함.
        2020-08-13T00:51:53.744059Z       169 Query     select parent0_.parent_id as parent_i1_1_, parent0_.name as name2_1_ from parent parent0_
        2020-08-13T00:51:53.786134Z       169 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=3
        2020-08-13T00:51:53.804123Z       169 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=2
        2020-08-13T00:51:53.810403Z       169 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=1
        */
        for (Parent parent : parentRepository.findAll()) {
            for (Child child : parent.getChildren()) {
                log.info("parent name : {}, child name : {}", parent.getName(), child.getName());
            }
        }
    }

    @Test
    @Transactional
    @Rollback(false)
    public void parentLazyChildLazyCollectionTest() throws Exception {
        /*
        지연 로딩 : N+1 문제가 발생함.
        2020-08-13T00:53:05.478201Z       179 Query     select parent0_.parent_id as parent_i1_1_, parent0_.name as name2_1_ from parent parent0_
        2020-08-13T00:53:05.539690Z       179 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=1
        2020-08-13T00:53:05.553336Z       179 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=2
        2020-08-13T00:53:05.556093Z       179 Query     select children0_.parent_id as parent_i3_0_0_, children0_.child_id as child_id1_0_0_, children0_.child_id as child_id1_0_1_, children0_.name as name2_0_1_, children0_.parent_id as parent_i3_0_1_ from child children0_ where children0_.parent_id=3
        */
        for (Parent parent : parentRepository.findAll()) {
            for (Child child : parent.getChildren()) {
                log.info("parent name : {}, child name : {}", parent.getName(), child.getName());
            }
        }
    }

    @Test
    @Transactional
    @Rollback(false)
    public void parentEagerChildEagerCollectionWithJoinFetchTest() throws Exception {
        /*
        JOIN FETCH 구문을 사용한 즉시 로딩 : N+1 문제가 해결됨.
        2020-08-13T00:55:12.907239Z       199 Query     select parent0_.parent_id as parent_i1_1_0_, children1_.child_id as child_id1_0_1_, parent0_.name as name2_1_0_, children1_.name as name2_0_1_, children1_.parent_id as parent_i3_0_1_, children1_.parent_id as parent_i3_0_0__, children1_.child_id as child_id1_0_0__ from parent parent0_ inner join child children1_ on parent0_.parent_id=children1_.parent_id
        */
        for (Parent parent : parentRepository.findAllJoinFetch()) {
            for (Child child : parent.getChildren()) {
                log.info("parent name : {}, child name : {}", parent.getName(), child.getName());
            }
        }
    }

    @Test
    @Transactional
    @Rollback(false)
    public void parentLazyChildLazyCollectionWithJoinFetchTest() throws Exception {
        /*
        JOIN FETCH 구문을 사용한 지연 로딩 : N+1 문제가 해결됨.
        2020-08-13T00:56:45.443285Z       209 Query     select child0_.child_id as child_id1_0_0_, parent1_.parent_id as parent_i1_1_1_, child0_.name as name2_0_0_, child0_.parent_id as parent_i3_0_0_, parent1_.name as name2_1_1_ from child child0_ inner join parent parent1_ on child0_.parent_id=parent1_.parent_id
        */
        for (Child child : childRepository.findAllJoinFetch()) {
            Parent parent = child.getParent();
            log.info("parent name : {}", parent.getName());
        }
    }


    @Test
    @Transactional
    @Rollback(false)
    public void parentEagerChildEagerCollectionWithEntityGraphTest() throws Exception {
        /*
        @EntityGraph 구문을 사용한 즉시 로딩 : N+1 문제가 해결됨.
        2020-08-13T00:58:44.113453Z       219 Query     select distinct parent0_.parent_id as parent_i1_1_0_, children1_.child_id as child_id1_0_1_, parent0_.name as name2_1_0_, children1_.name as name2_0_1_, children1_.parent_id as parent_i3_0_1_, children1_.parent_id as parent_i3_0_0__, children1_.child_id as child_id1_0_0__ from parent parent0_ left outer join child children1_ on parent0_.parent_id=children1_.parent_id
        */
        for (Parent parent : parentRepository.findAllByEntityGraph()) {
            for (Child child : parent.getChildren()) {
                log.info("parent name : {}, child name : {}", parent.getName(), child.getName());
            }
        }
    }

    @Test
    @Transactional
    @Rollback(false)
    public void parentLazyChildLazyCollectionWithEntityGraphTest() throws Exception {
        /*
        @EntityGraph 구문을 사용한 지연 로딩 : N+1 문제가 해결됨.
        2020-08-13T01:04:47.151227Z       239 Query     select distinct parent0_.parent_id as parent_i1_1_0_, children1_.child_id as child_id1_0_1_, parent0_.name as name2_1_0_, children1_.name as name2_0_1_, children1_.parent_id as parent_i3_0_1_, children1_.parent_id as parent_i3_0_0__, children1_.child_id as child_id1_0_0__ from parent parent0_ left outer join child children1_ on parent0_.parent_id=children1_.parent_id
        */
        for (Parent parent : parentRepository.findAllByEntityGraph()) {
            for (Child child : parent.getChildren()) {
                log.info("parent name : {}, child name : {}", parent.getName(), child.getName());
            }
        }
    }

    @Autowired
    ParentRepository parentRepository;

    @Autowired
    ChildRepository childRepository;
}
```

