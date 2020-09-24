---
title: Redis6 데이터 마이그레이션
comments: true
tags: [devops, docker, redis]
categories: [devops, redis]
header:
  teaser: "/assets/images/redis.png"
---
본 문서에서는 Redis 데이터 마이그레이션을 진행하는 방법에 대해서 설명하고 있습니다. <br/>

## Redis migration

### 사전 준비 과정

데이터 마이그레이션을 도커 기반으로 테스트 진행하기 위한

원본 redis6a, 타겟 redis6b 이미지를 다음과 같이 준비합니다.

```sh
# docker run -d -p 26379:6379 --name redis6a redis:6.0 --appendonly no --port 6379
# docker run -d -p 36379:6379 --name redis6b redis:6.0 --appendonly no --port 6379
```

<br/>

### 샘플 데이터 입력

##### redis6a

```sh
# docker exec -i -t redis6a redis-cli -h localhost -p 6379 
> set key1 value1
> set key3 value3
> save
> config get dir
1) "dir"
2) "/data"
```

<br/>

##### redis6b

```sh
# docker exec -i -t redis6b redis-cli -h localhost -p 6379 
> set key2 value2
> set key4 value4
> save
> config get dir
1) "dir"
2) "/data"
```

<br/>

### rump 설치

Redis 데이터 마이그레이션 툴인 [rump](https://github.com/stickermule/rump)를 설치합니다.

```sh
# curl -SL https://github.com/stickermule/rump/releases/download/1.0.0/rump-1.0.0-linux-amd64 -o rump \
  && chmod +x rump;
# ./rump --help
Usage of ./rump:
  -from string
        example: redis://127.0.0.1:6379/0 or /tmp/dump.rump
  -silent
        optional, no verbose output
  -to string
        example: redis://127.0.0.1:6379/0 or /tmp/dump.rump
  -ttl
        optional, enable ttl sync  
```



>AWS ElastiCache 또는 GCP MemoryStore Redis 클러스터 환경에서는 
>
>Redis의 표준 명령어인 BGSAVE과 SLAVEOF 명령어가 차단이 되기 때문에,
>
>데이터를 동기화하는 쉬운 방법은 없습니다.
>
><br/>
>
>하지만,
>
>Rump는 AWS / GCP Redis 클러스터 환경에서 사용가능한 SCAN, DUMP 및 
>
>RESTORE 명령어만을 사용하기 때문에, AWS / GCP Redis 클러스터 환경에서도 
>
>Redis의 데이터를 동기화하는데 사용할 수 있습니다.





<br/>

### rump를 사용한 데이터 마이그레이션

Elasticache 와 같은 클라우드 환경에서는 dump.rdb 기반으로 

데이터 마이그레이션을 진행할 수 없기때문에, rump와 같은 데이터 마이그레이션 툴을 사용해야 합니다.

```sh
# rump -from redis://127.0.0.1:26379 -to redis://127.0.0.1:36379
rrww
signal: exit
done

# docker exec -i -t redis6b redis-cli keys '*'
1) "key1"
2) "key3"
3) "key4"
4) "key2"
```

<br/>



### dump.rdb를 사용한 데이터 마이그레이션

데이터 마이그레이션 대상 서버의 appendonly 옵션이 no로 설정되어 있어야 합니다.

```sh
# docker cp redis6a:/data/dump.rdb .
# docker stop redis6b
# docker cp dump.rdb redis6b:/data/
# docker start redis6b
```

<br/>

### 데이터 마이그레이션 체크

기존에 redis6b가 가지고 있었던 key2, key4와 함께 key1, key3이 추가되었음을 확인합니다.

```sh
# docker exec -i -t redis6b redis-cli -h localhost -p 6379 
localhost:6379> keys *
1) "key1"
1) "key2"
1) "key3"
1) "key4"
```

<br/>

### appendonly 설정 복원

```sh
# docker exec -i -t redis6b redis-cli -h localhost -p 6379 
localhost:6379> config set appendonly yes
localhost:6379> quit
```

<br/>

### docker restart

```sh
# docker exec -i -t redis6a redis-cli -h localhost -p 6379            
localhost:6379> config get appendonly
1) "appendonly"
2) "yes"
localhost:6379localhost:6379> quit                                                              
```



