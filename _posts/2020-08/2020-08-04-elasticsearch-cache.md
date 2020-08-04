---
title: Elasticsearch 캐쉬
comments: true
tags: [devops, elasticsearch]
categories: devops
header:
  teaser: "/assets/images/elasticsearch.png"
---
본 문서에서는 Node 레밸 Query Cache와 Shared 레밸 Request Cache 에 대해서 설명하고 있습니다. <br/>

## 1. Node Query Cache



#### 1-1. Node Query Cache 의미 

Elasticsearch는 빈번하게 요청되는 filter 쿼리의 응답속도를 개선하기 위해서 캐쉬를 사용하며,

이 때, Node Query Cache가 적용이 됩니다.

<br/>

반면에, 그 때 그 때 score 계산이 필요한 query에는 Node Query Cache가 적용되지 않습니다.

<br/>

또한, Node Query Cache는 Query의 응답값만 캐쉬하고,

Query안에 포함될 수 있는 aggregation 과 같은 것들은 캐쉬를 하지 않습니다.

<br/>

#### 1-2. Node Query Cache 활성화

Node Query Cache 활성화 설정은 Elasticsearch cluster내의 모든 데이터 노드에 적용을 해야하며,

elasticsearch.yml 파일에 `index.queries.cache.enabled`을 설정하시면 됩니다. <br/>

* **`index.queries.cache.enabled`** : 활성화 유/무. (기본값 : true)

<br/>

#### 1-3. Node Query Cache Size 

Node Query Cache Size 설정은 Elasticsearch cluster내의 모든 데이터 노드에 적용을 해야하며,

elasticsearch.yml 파일에 `indices.queries.cache.size`을 설정하시면 됩니다. <br/>

* **`indices.queries.cache.size`** : JVM Heap의 몇 %를 캐쉬 공간으로 사용할지 설정함. (기본값 : 10%)

<br/>

## 2. Shard Request Cache

#### 2-1. Shared Request Cache 의미

Elasticsearch는 검색 요청이 인덱스 또는 여러 인덱스에 대해 실행될 때, 

관련된 각 샤드는 검색을 로컬 환경에서 실행하고 난 뒤의 결과를  coordinating node 로 반환하고, 

coordinating node는 각각의 결과를 수집하여, "글로벌"결과 집합으로 만들어 내는 과정으로 검색이 이루어지게 됩니다.

<br/>

해당 과정속에서 샤드 레벨 요청에 대한 로컬 응답값은 각 샤드의 로컬 캐시로 저장이 됩니다. 

(The shard-level request cache module caches the local results on each shard.)

<br/>

이 때, 만들어진 샤드의 로컬 캐쉬는 샤드가 변하는 순간 무효화되기 때문에,

Shared Request Cache는 변화가 거의 없는 정적인 데이터의 캐쉬환경에서 주로 사용이 됩니다.

<br/>

#### 2-2. Shared Request Cache 활성화

index 단위로 캐쉬 활성화를 설정 할 수도 있지만,

```json
# PUT /my-index-000001/_settings
{ 
    "index.requests.cache.enable": true 
}
```

쿼리 단위로도 캐쉬 활성화를 설정 할 수 있습니다.

```json
GET /my-index-000001/_search?request_cache=true
{
  "size": 0,
  "aggs": {
    "popular_colors": {
      "terms": {
        "field": "colors"
      }
    }
  }
}
```

이 때, size의 값이 0보다 큰 경우에는, 캐쉬되지 않습니다. 

<br/>

#### 2-3. Shard Request Cache Size 

캐쉬 사용 공간 설정은 다음과 같이 JVM Heap 공간에서 몇 %를 사용할지를 elasticsearch.yml 파일을 통해서 설정할 수 있습니다.

```yaml
indices.requests.cache.size: 2%
```

<br/>



## 3. Field Data Cache

#### 3-1. Field Data Cache의 의미

검색 결과에 포함된 문서를 빠르게 접근하기 위한 캐쉬이며, 주로 필드에서 집계를 정렬하거나 계산할 때, 사용되는 캐쉬 입니다.

필드 데이터 캐쉬는 메모리를 많이 사용할수도 있으므로, 사전에 충분한 메모리를 확보하는것이 좋습니다.

<br/>

#### 3-2. Field Data Cache 설정

캐쉬 사용 공간 설정은 elasticsearch.yml 파일을 통해서 설정할 수 있으며, 기본값은 무제한으로 설정이 되어 있습니다.

```yaml
indices.fielddata.cache.size : 
```

이 때, Field Data Cache의 사용공간을 제한하고 싶다면, `indices.breaker.fielddata.limit`설정을 하면 됩니다.

`indices.breaker.fielddata.limit`의 기본값은 40% 입니다.

```yaml
indices.breaker.fielddata.limit : 40%
```

또한, 특정 상황에서 약간의 메모리 오버헤드를 허용해 주고 싶을 때는, `indices.breaker.fielddata.overhead`를 설정하면 됩니다.

`indices.breaker.fielddata.overhead`의 기본값은 1.03이며, 해당 값을 `indices.breaker.fielddata.limit`에 곱한 값만큼 

메모리 오버헤드가 허용이 됩니다.

```yaml
indices.breaker.fielddata.overhead : 1.03
```



<br/>

## 4. Monitoring Cache

#### 4-1. Index별 Cache 통계

```sh
# GET /_stats/request_cache,query_cache?human&preety
```

<br/>

#### 4-2. Node별 Cache 통계

```sh
# GET /_nodes/stats/indices/request_cache,query_cache?human&preety
```

<br/>

#### 4-3. 특정 Index의 Cache 통계

```sh
# GET index_name/_stats/request_cache,query_cache?human&preety
```

<br/>

## 5. Clear cache

#### 5-1. 특정 Index의 Cache 삭제

```sh
# POST /my-index-000001/_cache/clear
# POST /my-index-000001,my-index-000002/_cache/clear
```

<br/>

#### 5-2. 모든 Index의 Cache 삭제

```sh
# POST /_cache/clear
```

<br/>

#### 5-3. 특정 Index의 특정 Cache 삭제

```sh
# POST /my-index-000001/_cache/clear?fielddata=true  
# POST /my-index-000001/_cache/clear?query=true      
# POST /my-index-000001/_cache/clear?request=true    
```

<br/>

#### 5-4. 특정 Index의 특정 field의 Cache 삭제

```sh
# POST /my-index-000001/_cache/clear?fields=foo,bar 
```

