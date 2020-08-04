---
title: Elasticsearch 샤드 축소 (shrink)
comments: true
tags: [devops, elasticsearch]
categories: devops
header:
  teaser: "/assets/images/elasticsearch.png"
---
## 1. Elasticsearch shard shrink 

index를 생성하는 시점에 설정되는 shard의 수를 변경할 때, _shrink 기능을 사용할 수 있습니다.

<br/>

#### 1-1. sample 색인 생성

```json
PUT sample
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 6
  },
  "mappings": {
    "properties": {
      "brand" : {
        "type": "keyword"
      }
    }
  }
}
```

<br/>

#### 1-2. sample 데이터 입력

```json
PUT sample/_doc/1
{
  "brand": "kia"
}

PUT sample/_doc/2
{
  "brand": "bmw"
}
```

<br/>



#### 1-3. sample 색인 정보 조회

```json
GET sample
{
  "sample" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "brand" : {
          "type" : "keyword"
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1594969085205",
        "number_of_shards" : "6",
        "number_of_replicas" : "0",
        "uuid" : "NDFd_VdbSwy8xAHcUZ549w",
        "version" : {
          "created" : "7020099"
        },
        "provided_name" : "sample"
      }
    }
  }
}
```

<br/>



#### 1-4. sample 색인을 Read only로 설정

shrink 될 색인 복제가 이루어지는 동안 sample 색인을 Read only 로 설정하고, <br/>

shrink 단계에서 복제되는 샤드는 `shrink_sample` 이름을 포함하게끔 설정합니다.

```json
PUT sample/_settings
{
  "settings": {
    "index.routing.allocation.require._name": "shrink_sample", 
    "index.blocks.write": true 
  }
}
```

<br/>



#### 1-5. sample 색인이 읽기전용으로 되었는지 확인

읽기전용이라서 색인 변경이 불가능함.

```json
PUT sample/_doc/2
{
  "brand": "bmw"
}
{
  "error": {
    "root_cause": [
      {
        "type": "cluster_block_exception",
        "reason": "index [sample] blocked by: [FORBIDDEN/8/index write (api)];"
      }
    ],
    "type": "cluster_block_exception",
    "reason": "index [sample] blocked by: [FORBIDDEN/8/index write (api)];"
  },
  "status": 403
}
```

<br/>



#### 1-6. sample 색인을 축소한 new_sample 색인을 생성

축소되어 새로 생성되는 색인의 shard 수는 원본 shard 수의 배수 또는 1로 설정해야 합니다.

```json
POST sample/_shrink/new_sample
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 2,
    "index.codec": "best_compression",
    "index.routing.allocation.require._name": null, 
    "index.blocks.write": null 
  }
}
```

<br/>

#### 1-7. new_sample 색인 조회

```json
GET new_sample
{
  "new_sample" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "brand" : {
          "type" : "keyword"
        }
      }
    },
    "settings" : {
      "index" : {
        "codec" : "best_compression",
        "routing" : {
          "allocation" : {
            "initial_recovery" : {
              "_id" : "SutN9vG7Trmx3q6FYn3o2w"
            },
            "require" : {
              "_name" : null
            }
          }
        },
        "allocation" : {
          "max_retries" : "1"
        },
        "number_of_shards" : "2",
        "routing_partition_size" : "1",
        "blocks" : {
          "write" : null
        },
        "provided_name" : "new_sample",
        "resize" : {
          "source" : {
            "name" : "sample",
            "uuid" : "a0H8NurmQZiJru8Nph5axw"
          }
        },
        "creation_date" : "1594969894561",
        "number_of_replicas" : "0",
        "uuid" : "ahAM2bk_R0a3NToUM8QLsA",
        "version" : {
          "created" : "7020099",
          "upgraded" : "7020099"
        }
      }
    }
  }
}
```











