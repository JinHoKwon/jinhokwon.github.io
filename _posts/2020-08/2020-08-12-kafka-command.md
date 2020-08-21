---
title: Kafka command 
tags: [devops, kafka]
comments: true
categories: kafka
header:
  teaser: "/assets/images/kafka/kafka_logo.png"
---

본 문서에서는 kafka 를 사용할 때, 자주 사용되는 명령어들에 대해서 설명하고 있습니다.<br/>

## 1. Topic

#### 1-1. topic 생성

기본값으로 생성시에 replication : 1, partition : 1로 설정이 됩니다.

```sh
$ kafka-topics.sh --bootstrap-server kafka1.net:9092 --create --topic testTopic
```

또는 다음과 같이 replication, partitions를 설정할 수 있습니다.

```sh
$ kafka-topics.sh --bootstrap-server kafka1.net:9092 --create --topic testTopic --partitions 2 --replication-factor 1
```

<br/>

#### 1-2. topic 수정

이미 생성된 topic은 다음과 같이 `--alter`명령어로 설정을 변경할 수 있습니다.

```sh
$ kafka-topics.sh --bootstrap-server kafka1.net:9092 --alter --topic testTopic --partitions 3
```

<br/>

#### 1-3. topic 리스트

```sh
$ kafka-topics.sh --bootstrap-server kafka1.net:9092 --list
__consumer_offsets
__transaction_state
testTopic
```

<br/>

#### 1-4. topic 조회

```sh
$ kafka-topics.sh --bootstrap-server kafka1.net:9092 --describe --topic testTopic
Topic: testTopic        PartitionCount: 3       ReplicationFactor: 1    Configs: min.insync.replicas=1,cleanup.policy=delete,flush.ms=1000,segment.bytes=1073741824,flush.messages=10000,max.message.bytes=20485760,unclean.leader.election.enable=false
        Topic: testTopic        Partition: 0    Leader: 1       Replicas: 1     Isr: 1
        Topic: testTopic        Partition: 1    Leader: 1       Replicas: 1     Isr: 1
        Topic: testTopic        Partition: 2    Leader: 1       Replicas: 1     Isr: 1
```

<br/>

#### 1-5. topic 삭제

```sh
$ kafka-topics.sh --bootstrap-server kafka1.net:9092 --delete --topic testTopic
```

또는 다음과 같이 **`--force`** 옵션을 사용하여 강제로 토픽을 삭제할 수 있습니다.

```sh
$ kafka-topics.sh --bootstrap-server kafka1.net:9092 --delete --topic testTopic --force
```



<br/>

## 2. Producer

#### 2-1. record 송신

```sh
$ kafka-console-producer.sh --bootstrap-server kafka1.net:9092 --topic testTopic
>sample1
>sample2
>sample3
```

그리고 재시도 간격을 5초로 설정할 수 있습니다.

```sh
$ kafka-console-producer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --producer-property "retry.backoff.ms=5000"
```

또한 acks 같은 경우 다음과 같이 설정할 수 있습니다.

```sh
$ kafka-console-producer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --producer-property "retry.backoff.ms=5000" \
--request-required-acks "all" --producer-property "transactional.id=777" --producer-property="enable.idempotence=true" 
```



<br/>

#### 2-2. record 송신 (트랜잭션)

```sh
$ kafka-console-producer.sh --bootstrap-server kafka1.net:9092 --topic testTopic  --producer-property enable.idempotence=true --request-required-acks all
```

<br/>

#### 2-3. key, value 송신

```sh
$ kafka-console-producer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --property "parse.key=true" --property "key.separator=:"
>mykey1:myvalue1
>mykey2:myvalue2
```

<br/>

## 3. Consumer

#### 3-1. record 수신

```sh
$ kafka-console-consumer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --from-beginning
```

그리고 재시도 간격을 5초로 설정할 수도 있습니다.

```sh
$ kafka-console-consumer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --from-beginning --consumer-property "retry.backoff.ms=5000"
```

또한 다음과 같이, 특정 Partion의 정해진 갯수만큼 메시지를 읽어들이는 것 또한 가능합니다.

```sh
$ kafka-console-consumer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --from-beginning --consumer-property "retry.backoff.ms=5000" --max-messages 3 --partition 0
$ kafka-console-consumer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --from-beginning --consumer-property "retry.backoff.ms=5000" --max-messages 3 --partition 1
$ kafka-console-consumer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --from-beginning --consumer-property "retry.backoff.ms=5000" --max-messages 3 --partition 2
```



<br/>

#### 3-2. consumer group 기반의 recrod 수신

```sh
$ kafka-console-consumer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --consumer-property group.id=test-consumer-group --consumer-property auto.offset.reset=earliest
sample3
sample1
sample2
```

<br/>

#### 3-3. 특정 consumer가 shutdown 된 경우

group.initial.rebalance.delay.ms=0 으로 설정된 경우,  곧바로 등록된 consumer가 제거되고, rebalance 가 진행됨.<br/>

group.initial.rebalance.delay.ms=0 이 아닌 경우, delay 시간 만큼 대기한 후 consumer가 제거되고, rebalance 가 진행됨.<br/>

<br/>

## 4. Consumer Group

#### 4-1.consumer group 리스트

```sh
$ kafka-consumer-groups.sh --bootstrap-server kafka1.net:9092 --all-groups --list
test-consumer-group
```

<br/>

#### 4-2. 특정 consumer group 출력

```sh
$ kafka-consumer-groups.sh --bootstrap-server kafka1.net:9092 --group test-consumer-group --describe
Consumer group 'test-consumer-group' has no active members.

GROUP               TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
test-consumer-group testTopic       0          1               1               0               -               -               -
test-consumer-group testTopic       1          1               1               0               -               -               -
test-consumer-group testTopic       2          1               1               0               -               -               -
```

<br/>

#### 4-3. consumer group offset 설정 

```sh
$ kafka-consumer-groups.sh --bootstrap-server kafka1.net:9092 --group test-consumer-group --topic testTopic --reset-offsets --to-datetime 2020-01-01T00:00:00.000 --dry-run
$ kafka-consumer-groups.sh --bootstrap-server kafka1.net:9092 --group test-consumer-group --topic testTopic --reset-offsets --to-offset 0 --dry-run

$ kafka-consumer-groups.sh --bootstrap-server kafka1.net:9092 --group test-consumer-group --topic testTopic --reset-offsets --to-datetime 2020-01-01T00:00:00.000 --execute
$ kafka-consumer-groups.sh --bootstrap-server kafka1.net:9092 --group test-consumer-group --topic testTopic --reset-offsets --to-offset 0 --execute
```

* **`--to-datetime`** : offset의 위치를 시간으로 설정함.
* **`--dry-run`** : 실제로 offset을 변경하지 않고, 계산된 offset 값만 출력함.
* **`--to-offset`** : offset의 위치를 position으로 설정함.



<br/>

#### 4-4. consumer group 삭제

```sh
$ kafka-consumer-groups.sh --bootstrap-server kafka1.net:9092 --group test-consumer-group --delete 
```

<br/>