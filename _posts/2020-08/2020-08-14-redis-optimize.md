---
title: Redis optimize
comments: true
tags: [devops, redis]
categories: redis
header:
  teaser: "/assets/images/redis.png"
---

Redis 최적화는 크게 시스템 최적화, Redis 설정 최적화로 구분되며,<br/>

본 문서에서는 일반적으로 잘 알려진 Redis 최적화 방법에 대해서 설명하고 있습니다.<br/>

## 1. Redis 를 위한 시스템 최적화 

#### 1-1. Max socket connection 

/etc/sysctl.conf 파일을 편집하여 최대 접속 가능한 Connection 수를 65365개로 설정합니다.

```sh
net.core.somaxconn=65365	# max connection (기본값 : 128개)
```

<br/>

#### 1-2. Over commit memory

over commit memory 관련 설정은 /etc/sysctl.conf 파일을 편집하여 설정할 수 있으며,<br/>

메모리가 부족한 상황속에서 다소 느리더라도 서비스가 계속 되어야 한다면, overcommit memory 설정을 1로 설정해 주어야 하고,<br/>

메모리가 부족한 상황이라면 서비스를 OOM으로 Kill 하는 편이 좋다고 판단되면, overcommit memory 설정을 0으로 설정해 주어야 합니다.

```sh
vm.overcommit_memory = 1	# enable overcommit
```

<br/>

#### 1-3. Transparent Huge Page(THP) 비활성화

 /etc/rc.local 파일에 아래 명령 줄을 추가

```sh
if test -f /sys/kernel/mm/transparent_hugepage/khugepaged/defrag; then
    echo 0 > /sys/kernel/mm/transparent_hugepage/khugepaged/defrag
fi

if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi

if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
    echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
```

이후 재부팅후 확인

```sh
# cat /sys/kernel/mm/transparent_hugepage/enabled
# cat  /sys/kernel/mm/redhat_transparent_hugepage/enabled
# cat /sys/kernel/mm/transparent_hugepage/defrag
# cat  /sys/kernel/mm/redhat_transparent_hugepage/defrag
```



 <br/>

## 2. Redis 최적화

본 색션에서는 redis.conf 파일을 수정하여 Redis를 최적화하는 방법에 대해서 설명하고 있습니다.<br/>



#### 2-1. tcp-keepalive

클라이언트가 죽었을때 레디스 서버가 확인해서 클라이언트와의 접속을 제거하는 시간입니다. (단위 : 초)<br/>

기본값은 **300**입니다.<br/>

<br/>

#### 2-2. maxclient

최대 접속자수를 설정합니다. <br/>

기본값은 **10000**입니다.

<br/>



#### 2-3. maxmemory

최대 사용 가능한 메모리를 설정하며, 보통 물리메모리의 약 70% 정도를 할당합니다. <br/>

0 으로 설정시 unlimited이며, 기본값은 **0**입니다.<br/>

<br/>

#### 2-4. maxmemory-policy

최대 사용 가능한 메모리 한계 도달시 처리 정책을 설정합니다.<br/>

- **volatile-lru** : 만료 시간이 설정된 키중에서 근사 LRU로 삭제할 키를 정한다.
- **allkeys-lru** : 모든 키중에서 근사 LRU로 삭제할 키를 정한다.
- **volatile-lfu** : 만료 시간이 설정된 키중에서 근사 LFU로 삭제할 키를 정한다.
- **allkeys-lfu** : 모든 키중에서 근사 LFU로 삭제할 키를 정한다.
- **volatile-random** : 만료 시간이 설정된 키중에서 임의(random)로 삭제할 키를 정한다.
- **allkeys-random** : 모든 키중에서 임의로 삭제할 키를 정한다.
- **volatile-ttl** : 만료시간이 가장 가까운 키 순으로 삭제한다.
- **noeviction** : 키를 삭제하지 않는다. 쓰기 명령에 에러를 리턴한다.

<br/>

기본값은 **noeviction** 입니다.<br/>

<br/>

#### 2-5. Pipelining

Redis의 명령어들을 한꺼번에 일괄 처리하고자 할 때 Pipelining을 사용할 수 있습니다.

```java
@Test
@Order(1)
public void pipelineTest() {
    this.stringRedisTemplate.executePipelined(new RedisCallback<Object>() {
        @Override
        public Object doInRedis(RedisConnection connection) throws DataAccessException {
            StringRedisConnection stringRedisConnection = (StringRedisConnection)connection;
            stringRedisConnection.set("key1", "value1");
            stringRedisConnection.set("key2", "value2");
            stringRedisConnection.set("key3", "value3");
            return null;
        }
    });
}
```
##### 실행 결과 확인

```sh
# redis-cli -h localhost -p 6379
localhost:6379> keys key*
1) "key1"
2) "key3"
3) "key2"
```







<br/>

