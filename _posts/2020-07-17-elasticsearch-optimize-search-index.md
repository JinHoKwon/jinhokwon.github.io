---
title: Elasticsearch 검색, 색인 최적화
comments: true
tags: [devops, elasticsearch]
categories: devops
header:
  teaser: "/assets/images/elasticsearch.png"
---
## 1. 검색 속도 최적화

#### 1-1. 분산 처리를 하기 위해서 적절한 Shard의 갯수를 선정함.

모든 상황을 만족하는 Shard의 갯수는 없기 때문에,

검색과 색인시점에 적절한 샤드의 갯수를 찾는 과정은 매우 중요합니다. 

<br/>

다만, 일반적으로는 <br/>

Primary Shard 수가 증가하면 indexing은 빨라지고, 검색 성능은 저하되며,

Replica Shard 의 수가 증가하면, indexing은 느려지고 검색 성능은 빨라집니다.

<br/>

각각 trade off 되는 부분을 잘 이해할 필요가 있습니다.

<br/>

그리고, Failover를 자동으로 대응하기 위해서는 

적절한 Replica Shard의 수 (최대 Replica Shared의 수 : `전체 노드의 수 -1`) 를 적절하게 선정하는것 또한 중요합니다.

<br/>

**색인 배치 만들때 참고사항**

> Primary shard의 수는 색인을 생성하는 시점 이후로 변경이 불가능하지만,
>
> Replica shard의 수는 언제든지 변경이 가능합니다.
>
> 그래서 보통은 색인 배치가 실행되는 시점에 Replica 를 Disable 하고 
>
> 색인 배치가 완료된 이후에, 적절한 Replica 수로 설정합니다.

<br/>

#### 1-2. 검색 결과가 캐쉬되는 필터를 우선 사용

Keyword 검색 방식인 필터는 검색 결과가 캐쉬되지만,<br/>
Full Text Search 검색 방식인 쿼리는 검색 결과가 캐쉬가 안되기 때문에,<br/>
점수 계산 보다 성능이 중요한 경우에는 쿼리 보다는 필터 검색을 우선 고려해야 합니다.
<br/>

<br/>

#### 1-3. 페이징 처리

5개의 샤드를 운영중이고 1페이지당 50개의 문서를 보여준다고 했을 때<br/>
1페이지를 보기 위해 가져오는 문서의 갯수 = 5 * 50 * 1<br/>
2페이지를 보기 위해 가져오는 문서의 갯수 = 5 * 50 * 2 <br/>
...<br/>
50페이지를 보기 위해 가져오는 문서의 갯수 = 5 * 50 * 50 <br/>

<br/>페이지가 뒤로 갈수록 리소스 부담이 늘어나는 특징을 잘 고려해야 합니다.

<br/>
<br/>

#### 1-4.  doc['field']을 쓰는게 좋은 이유
_source, _field는 디스크 접근 방식이지만, doc['field']는 캐시 접근 방식이라서,<br/>
가급적, doc['field'] 방식으로 사용하는것이 좋습니다.

<br/>

#### 1-5. Search Type의 이해

* Query and fetch : Score, Ranking작업을 진행하지 않습니다.
* Query then fetch : Score, Ranking작업을 진행하기에 속도가 느립니다.



<br/>

<br/>

## 2. 색인 최적화

indexing을 최적화 하기 위해서는 색인이 필요한 문서를 Bulk 단위로 요청하는 것이 좋습니다.<br/>
이 때 적절한 요청 크기는 1,000~5,000개의 도큐먼트나 5~15MB 사이의 물리적 크기가 좋습니다.
<br/>

<br/>

#### 2-1. 벌크 indexing 과정

* Replica, Refresh 설정을 disable 합니다.
* 인덱싱할 문서를 읽는다.
* Bulk 요청
* 최적화 작업 수행 (refresh)
* Replica, Refresh 설정을 복구한다.
<br/>
<br/>

#### 2-2. 적절한 샤드 수 및 크기

정해진 공식같은 것은 없으나, 가급적 다음에 언급되고 있는 최대치 이하로 관리하는 것이 좋습니다.
```
A good rule of thumb is to try to keep shard size between 10–50 GiB. Large shards can make it difficult for Elasticsearch to recover from failure, but because each shard uses some amount of CPU and memory, having too many small shards can cause performance issues and out of memory errors.
```

* Index당 최대 Shard 수 : 200개 이하
* Shard 하나 당 최대 크기 : 10~50GB
<br/>
<br/>