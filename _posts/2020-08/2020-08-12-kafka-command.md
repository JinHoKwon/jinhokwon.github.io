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

<br/>

## 2. Producer

#### 2-1. record 송신

```sh
$ kafka-console-producer.sh --bootstrap-server kafka1.net:9092 --topic testTopic
>sample1
>sample2
>sample3
```

<br/>

#### 2-2. record 송신 (트랜잭션)

```sh
$ kafka-console-producer.sh --bootstrap-server kafka1.net:9092 --topic testTopic  --producer-property enable.idempotence=true --request-required-acks all
```

<br/>

## 3. Consumer

#### 3-1. record 수신

```sh
$ kafka-console-consumer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --from-beginning
```

<br/>

#### 3-2. consumer group 기반의 recrod 수신

```sh
$ kafka-console-consumer.sh --bootstrap-server kafka1.net:9092 --topic testTopic --consumer-property group.id=test-consumer-group --consumer-property auto.offset.reset=earliest
```

<br/>

## 4. Consumer Group