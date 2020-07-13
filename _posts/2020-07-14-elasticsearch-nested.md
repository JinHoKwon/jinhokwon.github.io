---
title: Elasticsearch object vs nested
tags: [devops, elasticsearch]
categories: devops
header:
  teaser: "/assets/images/elasticsearch.png"
---
Elasticsearch object 타입은 object 타입내에 포함된 properties 에 대한 연관성 정보가 없지만,
Elasticsearch nested 타입은 nested 타입내에 포함된 properties 에 대한 연관성 정보가 있습니다.

즉, 연관성 정보가 없는 object 타입으로 properties 를 입력하게 될 경우,

```json
PUT objectcars/_doc/1
{
  "brand": "kia",
  "model": [
    {
      "name": "k5",
      "year": 2010
    },
    {
      "name": "pride",
      "year": 2011
    }
  ]
}
```
다음과 같이 해석됩니다.

model.name : ["k5", "pride"]
model.year : [2010, 2011] 

반면에, 동일한 문서를 nested 타입으로 입력하게 될 경우,
model 의 하위 properties 는 별도의 hidden document 로 색인이 됩니다.  

이와 같은 특징으로, object 타입으로 properties 를 검색하게 될 경우
properties 중의 한개만 포함이 되어있어도 검색이 되며,
properties의 개별 검색은 불가능합니다. 

이 때, 

개별 properties 에 대한 개별 검색까지 가능하게 하려면
nested 타입으로 mapping 을 해주어야 합니다. 

그리고, 

nested 타입으로 mapping 을 하게 될 경우,
properties 의 하위 문서 또한 모두 다 색인하기 때문에,
그 만큼 공간을 더 많이 차지한다는 단점이 있습니다. 



#### object 타입 사용 예제

##### object 타입 샘플 색인 생성

```json
PUT objectcars
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "brand" : {
        "type": "keyword"
      },
      "model" : {
        "type" : "object",
        "properties": {
          "name" : {
            "type" : "text"
          },
          "year" : {
            "type" : "integer"
          }
        }
      }
    }
  }
}
```



##### object 타입 샘플 데이터 입력

```json
PUT objectcars/_doc/1
{
  "brand": "kia",
  "model": [
    {
      "name": "k5",
      "year": 2010
    },
    {
      "name": "pride",
      "year": 2011
    }
  ]
}
```



##### object 타입 샘플 데이터 검색

k5 는 2010년에 생성되었지만, object 타입에는 연관성 정보가 없기 때문에,

model.name 또는 model.year 에 기록된 배열값중 한개만 만족하면 검색이 됩니다. 

즉, model 이라는 document 단위가 아닌, model.name 배열과 model.year 배열 단위로 검색됩니다.

```json
GET objectcars/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "model.name": {
              "value": "k5"
            }
          }
        },
        {
          "term": {
            "model.year": {
              "value": 2011
            }
          }
        }
      ]
    }
  }
}

{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.287682,
    "hits" : [
      {
        "_index" : "objectcars",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.287682,
        "_source" : {
          "brand" : "kia",
          "model" : [
            {
              "name" : "k5",
              "year" : 2010
            },
            {
              "name" : "pride",
              "year" : 2011
            }
          ]
        }
      }
    ]
  }
}
```



#### nested 타입 사용 예제



##### nested 타입 샘플 색인 생성

```json
PUT nestedcars 
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "brand" : {
        "type": "keyword"
      },
      "model" : {
        "type" : "nested",
        "properties": {
          "name" : {
            "type" : "text"
          },
          "year" : {
            "type" : "integer"
          }
        }
      }
    }
  }
}
```



##### nested 타입 샘플 데이터 입력

```json
PUT nestedcars/_doc/1
{
  "brand": "kia",
  "model": [
    {
      "name": "k5",
      "year": 2010
    },
    {
      "name": "pride",
      "year": 2011
    }
  ]
}
```



##### nested 타입 샘플 데이터 검색

nested 타입은 각각의 model 을 document (연관성 정보가 포함됨.) 처럼 관리하기 때문에 

model.name 과 model.year 정보를 model 단위로 검색할 수 있습니다.

즉, 다음과 같이 pride, 2010 과 같이 검색하면 검색 결과 없음이 나오지만,

pride, 2011 과 같이 검색하면 검색 결과를 확인할 수 있습니다.

```json
GET nestedcars/_search
{
  "query": {
    "nested": {
      "path": "model",
      "query": {
        "bool": {
          "must": [
            {
              "term": {
                "model.name": {
                  "value": "pride"
                }
              }
            },
            {
              "term": {
                "model.year": {
                  "value": 2010
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

