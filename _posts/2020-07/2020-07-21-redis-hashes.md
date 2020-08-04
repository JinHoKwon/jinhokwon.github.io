---
title: Redis hashes
tags: [devops, redis]
comments: true
categories: redis
header:
  teaser: "/assets/images/redis.png"
---



## 1. Redis hashes 사용시 주의사항

Redis cluster 환경에서는 Primary redis node 와 slave redis node 간의 

동기화가 100% 이루어지기 이전까지는, 해쉬 정보가 서로 다를수 있습니다.

<br/>

## 2. Redis hash 이해



Redis hash는 1개의 key에 n개의 field, value를 저장할 수 있습니다. 

이 때 저장 가능한 최대 field는 약 40억개 정도 입니다.

> Every hash can store up to 232 - 1 field-value pairs (more than 4 billion)



#### 2-1. Redis 서버 실행 및 초기화

```sh
# bin/redis-server conf/redis.conf
# redis-cli -h localhost -p 6379 
localhost:6379> flushall
localhost:6379> OK
```

<br/>

#### 2-2. hset : field 추가

> hset <key> <field1> <value1> <field2> <value2> ... <field N> <value N>  

bbs 키에 ebs, mbc, kbs 맴버를 추가하고 각각 1로 설정합니다.

```sh
localhost:6379> hset bbs ebs 1 mbc 1 kbs 1 ex 10
(integer) 4
```

<br/>

#### 2-3. hgetall : 모든 field 조회

> hgetall <key> 

bbs 키 값에 매핑된 모든 field를 조회합니다.
```sh
localhost:6379> hgetall bbs
1) "ebs"
2) "1"
3) "mbc"
4) "1"
5) "kbs"
6) "1"
7) "ex"
8) "10"
```

<br/>

#### 2-4. hmget : 여러개의 value를 조회

> hmget <key> <field1> <field2> ... <fieldN>

bbs 해쉬의 ebs, mbc, kbs의 값을 조회합니다.

```sh
localhost:6379> hmget bbs ebs mbc kbs
1) "1"
2) "1"
3) "1"
```

<br/>

#### 2-5. hdel : 필드 삭제

> hdel <key> [field ...]

bbs 해쉬의 포함된 ebs 맴버를 삭제합니다.

```sh
localhost:6379> hdel bbs ebs
(integer) 1

localhost:6379> hgetall bbs
1) "mbc"
2) "1"
3) "kbs"
4) "1"
```

<br/>

#### 2-6. hmset : 여러개의 field, value를 설정

> hmset <key> <field1> <value1>  ... <field N> <value N>

bbs 해쉬의 mbc 는 11, kbs 7 로 설정합니다.

```sh
localhost:6379> hmset bbs mbc 11 kbs 7
OK

localhost:6379> hgetall bbs
1) "mbc"
2) "11"
3) "kbs"
4) "7"
```

<br/>

#### 2-7. hincrby : field 값을 increment 값 만큼 더함

> hincrby  <key> <field> <increment>

mbc의 값을 1만큼 증가하고, kbs의 값을 5만큼 증가

```sh
localhost:6379> hincrby bbs mbc 1
(integer) 12

localhost:6379> hincrby bbs kbs 5
(integer) 12
```

