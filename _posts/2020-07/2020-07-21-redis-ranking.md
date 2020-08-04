---
title: Redis ranking
tags: [devops, redis]
comments: true
categories: redis
header:
  teaser: "/assets/images/redis.png"
---



## 1. Redis ranking 사용시 주의사항

Redis cluster 환경에서는 Primary redis node 와 slave redis node 간의 

동기화가 100% 이루어지기 이전까지는, 랭킹 정보가 서로 다를수 있습니다.

<br/>

## 2. Redis Ranking 이해



#### 2-1. Redis 서버 실행 및 초기화

```sh
# bin/redis-server conf/redis.conf
# redis-cli -h localhost -p 6379 
localhost:6379> flushall
localhost:6379> OK
```

<br/>

#### 2-2. zadd : 맴버 추가

> zadd <key> <score> <member> [score member ...]

bbs 키에 ebs, mbc, kbs, sbs 맴버를 추가하고 각각 1점씩 부여합니다.

이 때, 이미 존재하는 맴버라면, 입력된 점수로 덮어쓰기를 합니다.

```sh
localhost:6379> zadd bbs 1 ebs 1 mbc 1 kbs 1 sbs
(integer) 4
```

<br/>

#### 2-3. zrevrange : 순위조회

> zrange <key> <시작> <끝> withscores

bbs 키 값에 매핑된 모든 맴버의 점수를 포함하여 조회합니다.
```sh
localhost:6379> zrevrange bbs 0 -1 withscores
1) "sbs"
2) "1"
3) "mbc"
4) "1"
5) "kbs"
6) "1"
7) "ebs"
8) "1"
```

<br/>

#### 2-4. zincr : 점수 추가

> zincr bbs <score> <member>

kbs에 +2점을 추가합니다.

```sh
localhost:6379> zincrby bbs 2 kbs
localhost:6379> "3"

localhost:6379> zrevrange bbs 0 -1 withscores
1) "kbs"
2) "3"
3) "sbs"
4) "1"
5) "mbc"
6) "1"
7) "ebs"
8) "1"
```

<br/>

#### 2-5. zrem : 맴버 삭제

> zrem <key> [member ...]

bbs 키값에 포함된 ebs 맴버를 삭제합니다.

```sh
localhost:6379> zrem bbs ebs
(integer) 1

localhost:6379> zrevrange bbs 0 -1 withscores
1) "kbs"
2) "3"
3) "sbs"
4) "1"
5) "mbc"
6) "1"
```

<br/>

#### 2-6. zcount : 점수 기준 맴버 수 조회

> zcount <key> <최소점수> <최대점수> 

이 때, 음의 무한 정수는 `-inf` 로 표시하고, 양의 무한 정수는 `+inf`로 표시합니다.

```sh
localhost:6379> zcount bbs -inf +inf
(integer) 3

localhost:6379> zcount bbs 2 +inf
(integer) 1
```

<br/>

#### 2-7. zrevrangebyscore : 점수를 지정하여 랭킹을 조회

> zrevrangebyscore  <key> <최소점수> <최대점수>

최대 3점을 포함한 3점 이하까지 조회

```sh
localhost:6379> zrevrangebyscore bbs 3 -inf
1) "kbs"
2) "sbs"
3) "mbc"
```

최대 3점 이하 조회

```sh
localhost:6379> zrevrangebyscore bbs (3 -inf
1) "sbs"
2) "mbc"
```

<br/>

#### 2-8. zrevrank : 맴버 랭킹 조회

> zrevrank <key> <맴버>

mbc, kbs의 랭킹을 조회합니다.

```sh
localhost:6379> zrevrank bbs mbc
(integer) 2

localhost:6379> zrevrank bbs kbs
(integer) 0
```

이 때, 반환값 +1을 해주어야 올바른 등수가 됩니다.