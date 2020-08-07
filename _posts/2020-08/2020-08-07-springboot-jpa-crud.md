---
title: SpringBoot JPA CRUD
tags: [springboot, dev]
comments: true
categories: springboot
header:
  teaser: "/assets/images/springboot/springboot_logo.png"
---

본 문서에서는 JPA를 활용한 CRUD 방법에 대해서 소개하고 있습니다. <br/>

## 1. JPA CRUD 

#### 1-1. Box.java

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

위와 같이 선언된 `Box` Entity는 다음과 같은 `CREATE TABLE`구문으로 매핑이 됩니다.

```sql
CREATE TABLE `box` (
	`id` BIGINT(19,0) NOT NULL AUTO_INCREMENT,
	`code` VARCHAR(64) NULL DEFAULT NULL COLLATE 'utf8_general_ci',
	`name` VARCHAR(64) NULL DEFAULT NULL COLLATE 'utf8_general_ci',
	PRIMARY KEY (`id`) USING BTREE,
	UNIQUE INDEX `UK_eaj689yduj53wefw3qtj6gdeb` (`code`) USING BTREE
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
AUTO_INCREMENT=2
;
```

<br/>

#### 1-2. BoxRepository.java

```java
package com.example.app;

import com.example.app.model.Box;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface BoxRepository extends JpaRepository<Box, Long> {

}
```

<br/>

#### 1-3. JPA CRUD

```java
@Test
public void crudTest() throws Exception {
    Box box = new Box("red", "001");
    Box saveBox = boxRepository.save(box);
    Box readBox = boxRepository.findById(saveBox.getId()).get();
    readBox.setCode("002");
    Box updateBox = boxRepository.save(readBox);
    boxRepository.deleteById(updateBox.getId());
}

@Autowired
BoxRepository boxRepository;
```



<br/>



