---
title: SpringBoot JPA @OneToMany
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

본 문서에서는 JPA를 활용한 @OneToMany, @ManyToOne 어노테이션 소개 및 사용방법에 대해서 소개하고 있습니다. <br/>

## 1. JPA @OneToMany, @ManyToOne

#### 1-1. 관계 정의

* @OneToMany : 일대다 관계
* @ManyToOne : 다대일 관계 
* 항상 **Many(다)**쪽이 관계의 주인이 되며, 그 반대편이 하인이 됨. 
* mappedBy 속성으로 외래키를 설정
* ```java
  // Parent와 Child는 1:N의 관계이기 때문에, 관계의 주인은 N쪽에 위치한 Child 입니다.
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
  ```

* @OneToMany, @ManyToOne 사용시에, JPA N+1 문제를 주의해야 합니다.

<br/>

#### 1-2. 데이터 I/O

* 데이터를 추가할 때는 하인 > 주인 순으로 진행
* 데이터를 삭제해야 할 때는 주인 > 하인순으로 진행

만약, 순서가 어긋난 경우에는 `ConstraintViolationException `이 발생합니다.

<br/>

#### 1-3. 단방향, 양방향 설정

마지막으로 JPA 관계는 단방향으로 할지 양방향으로 할지 결정할 수 가 있는데,

JPA @OneToMany 같은 경우 단방향으로 설계시 다음과 같은 단점이 있습니다.

- 엔티티가 관리하는 외래 키가 다른 테이블에 있음 (Many에 외래키 존재)
- 연관관계 관리를 위해 추가로 update sql 실행 (성능상 큰 차이는 없다)
- 개발을 하다 보면 B를 만졌는데 A도 update sql문이 나가니 헷갈린다.
- 그래서 필요하다면 일대다 보다는 양방향 관계로 한다. ( B는 A가 필요 없더라도, 객체 지향적으로 손해를 보는 거 같지만) - 트레이드 오프

(출처) https://dublin-java.tistory.com/51

이러한 이유로, 단방향으로 할지, 양방향으로 할지 판단하기 모호한 경우, `양방향`으로 설계하는편이 좋습니다.



<br/>

지금까지 일반적으로 알고있어야 하는 내용들에 정리해 보았습니다.

<br/>





## 2. JPA @OneToMany, @ManyToOne



본 색션에서는 관계를 다룰때, 가장 많이 등장하는 Parent (1) : Child (N) 관계 및 Child (N) : Parent(1)를 양방향으로 구현하고,<br/>

JPA 환경에서 어떻게 적용하는지에 대한 예제에 대해서 소개하고 있습니다.<br/>

<br/>

[여기](/springboot/springboot-jpa-dev-setting/)를 참고하여, JPA 개발환경 구축을 진행하고, 다음 클래스들을 추가합니다.

<br/>

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

    // 관계의 주인의 Many(다)쪽인 Child 이기 때문에, 
    // mappedBy 속성을 Parent 클래스에 적용합니다. 
    // 이 때 입력되는 속성값은 관계의 주인인 Child 클래스에서 선언한 Parent 클래스의 변수명이 됩니다.
    
    // 추가로, Many(다)쪽을 저장하는 Collection 을 Set으로 한 이유는 JPA 환경에서 발생할 수 있는 n+1 문제를 
    // 방지하기 위해서 Set을 사용합니다. (나중에 n+1 문제 소개할 때 다시 한 번 더 언급합니다.)
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
    @NonNull
    Set<Child> children = new LinkedHashSet<>();

    // 관계를 별도의 테이블로 분리하여 관리하기 위해서 @JoinTable 어노테이션을 사용함.
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinTable(
            name = "parent_car",                                  // 매핑할 조인 테이블
            joinColumns = @JoinColumn(name = "parent_id"),        // 현재 엔티티를 참조하는 외래 키
            inverseJoinColumns = @JoinColumn(name = "car_id")     // 반대방향 엔티티를 참조하는 외래 키
    )
    List<Car> cars = new ArrayList<>();

    @Override
    public String toString() {
        return id + ":" + name + ":" + children.size();
    }
}
```

위와 같이 선언된 `Parent` Entity는 다음과 같은 `CREATE TABLE`구문으로 매핑이 됩니다.

```sql
CREATE TABLE `parent` (
    `parent_id` BIGINT(19,0) NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(64) NULL DEFAULT NULL COLLATE 'utf8_general_ci',
    PRIMARY KEY (`parent_id`) USING BTREE,
    UNIQUE INDEX `UK_tkff6d2q9uuhuhs0gya31w046` (`name`) USING BTREE
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB
AUTO_INCREMENT=2
;
```

그리고, @JoinTable 어노테이션으로 선언한 부분은 다음과 같은 `CREATE TABLE` 구문으로 매핑이 됩니다.

```sql
CREATE TABLE `parent_car` (
    `parent_id` BIGINT(19,0) NOT NULL,
    `car_id` BIGINT(19,0) NOT NULL,
    UNIQUE INDEX `UK_ot0c1u57e5bv4w8l0347iv9f3` (`car_id`) USING BTREE,
    INDEX `FKkl1y0iotd4nv8wm13ovi573pp` (`parent_id`) USING BTREE,
    CONSTRAINT `FKkl1y0iotd4nv8wm13ovi573pp` FOREIGN KEY (`parent_id`) REFERENCES `test`.`parent` (`parent_id`) ON UPDATE NO ACTION ON DELETE NO ACTION,
    CONSTRAINT `FKkq7g2csxtdqlo1ytimufcfhim` FOREIGN KEY (`car_id`) REFERENCES `test`.`car` (`car_id`) ON UPDATE NO ACTION ON DELETE NO ACTION
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB
;
```



<br/>

#### 2-2. ParentRepository.java

ParentRepository 클래스에서는 N+1 문제를 해결하기 위해서, `JOIN FETCH` 방식과

`@EntityGraph` 방식을 함께 적용해 두었습니다. 

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

    // N+1 문제를 해결하기 위한 JOIN FETCH
    @Query("SELECT parent FROM Parent parent JOIN FETCH parent.children")
    Set<Parent> findAllJoinFetch();

    // N+1 문제를 해결하기 위한 @EntityGraph
    @EntityGraph(attributePaths = {"children"})
    @Query("SELECT DISTINCT parent FROM Parent parent")
    List<Parent> findAllByEntityGraph();
}
```

<br/>

#### 2-3. Child.java

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

위와 같이 선언된 `Child` Entity는 다음과 같은 `CREATE TABLE`구문으로 매핑이 됩니다.<br/>

Child Entity는 관계의 주인이기 때문에, FOREIGN KEY 키가 포함이 됩니다.

```sql
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
AUTO_INCREMENT=3
;
```

<br/>

#### 2-4. ChildRepository.java

```java
package com.example.app.repository;

import com.example.app.model.Child;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ChildRepository extends JpaRepository<Child, Long> {
    Child findByName(String name);
}
```

<br/>

#### 2-5. Car.java

Car Entity는 관계의 주인이 없는 대신, Parent Entity에 @JoinTable을 사용하여 관계를 설정하고 있습니다.

즉, Car 입장에서는 소유자 (Parent)를 모르고 있습니다. 

```java
package com.example.app.model;

import lombok.*;

import javax.persistence.*;

@Entity
@Table(name = "car")
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PUBLIC)
@ToString(of = {"id", "name"})
@RequiredArgsConstructor
public class Car {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name="car_id")
    long id;

    @Column(name="name")
    @NonNull
    String name;
}
```

위와 같이 선언된 `Car` Entity는 다음과 같은 `CREATE TABLE`구문으로 매핑이 됩니다.

```sql
CREATE TABLE `car` (
    `car_id` BIGINT(19,0) NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(255) NULL DEFAULT NULL COLLATE 'utf8_general_ci',
    PRIMARY KEY (`car_id`) USING BTREE
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB
;
```

<br/>



#### 2-6. CarRepository.java

```java
package com.example.app.repository;

import com.example.app.model.Car;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface CarRepository extends JpaRepository<Car, Long> {
    Car findByName(String name);
}
```

<br/>

#### 2-7. JPA @OneToMany Test

```java
package com.example.app;

import com.example.app.model.Car;
import com.example.app.model.Child;
import com.example.app.model.Parent;
import com.example.app.repository.CarRepository;
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

import java.util.*;

@Slf4j
@ExtendWith(SpringExtension.class)
@SpringBootTest
public class SpringBootConsoleApplicationTests {

    @Test
    @Transactional
    @Rollback(false)
    public void oneToManyTest() throws Exception {
        Parent parent = new Parent("father");
        Set<Child> children = new LinkedHashSet<>();

        children.add(new Child("james", parent));
        children.add(new Child("tomas", parent));
        parent.setChildren(children);

        Parent dbParent = parentRepository.save(parent);
        dbParent.getChildren().removeIf(child -> child.getName().equalsIgnoreCase("james"));
    }
}
```

##### 실행결과

```sql
SET autocommit=0
insert into parent (name) values ('father')
insert into child (name, parent_id) values ('james', 1)
insert into child (name, parent_id) values ('tomas', 1)
delete from child where child_id=1
commit
SET autocommit=1
```

