# Elasticsearch

## 주요 용어
- 색인(indexing) : 데이터가 검색될 수 있는 구조로 변경하기 위해 원본 문서를 검색어 토큰들로 변환하여 저장하는 일련의 과정
- 인덱스(index, indices) : 색인 과정을 거친 결과물 또는 색인된 데이터가 저장되는 저장소
- 도큐먼트(document) : 단일 데이터 단위
- 검색(search) : 인덱스에 들어있는 검색어 코튼들을 포함하고 있는 문서를 찾아가는 과정
- 질의(Query) : 사용자가 원하는 문서를 찾거나 집계 결과를 출력하기 위해 검색 시 입력하는 검색어 또는 검색 조건
- 샤드(Shards) : 인덱스를 쪼게는 단위
- 매핑(Mapping) : 도큐먼트 내부 데이터명. RDB의 칼럼 개념
- 디스커버리(Discovery) : 노드가 처음 실행 될 때 같은 서버 또는 임의로 설정된 네트워크 상의 다른 노드들을 찾아 하나의 클러스터로 바인딩하는 과정

## 엘라스틱처치는
- Apach Lucene 기반 (Written in Java)
- 자바로 개발되어있 때문에 자바 실행이 가능한 환경이라면 어디서든 구동이 가능
- inverted index 구조로 데이터 저장
    - RDBMS와 반대 구조
    - 텍스트를 파싱해서 검색어 사전을 만든 다음에 inverted index 방식으로 텍스트를 저장
- 텍스트 처리과정
    1) 대소문자 변환
    2) 토큰 재정렬(보통 ascii 순서로)
    3) 불용어(stopwords, 검색어로서의 가치가 없는 단어들) 제거. a, an, be, the, to 등
    4) 토큰 병합 (e.g. jumping, jumps가 jump로 병합)
    5) 동의어 처리 (e.g. fast 7 quick)
- 검색 과정
    - 검색어도 동일하게 텍스트 처리를 하여 검색
- 일반적으로 맵리듀스 처리 방법을 이용해 액션을 수행한다.
- 엘라스틱서치 노드에는 두 가지 중요한 노드가 있다.
    - Data Node
        - 데이터를 저장하며 색인된 도큐먼트를 저장한 색인 샤드를 포함한다.
    - Non-data Node (결정권자)
        - Rest 응답을 비롯한 그 밖의 모든 검색 작업을 처리
        - 일반적으로 엘라스틱서치는 맵리듀스 처리 방법을 이용하여 액션을 처리한다.
        - 기본 샤드에 액션을 분산하고, 최종 응답을 보낼 수 있도록 샤드 결과 수집/집계(리듀스)를 책임진다.
        - 패싯(facet), 집계, 캐싱 같은 작업을 하기 위해 많은 양의 RAM을 사용할 수 있다.

## Protocol
엘라스틱서치는 여러 프로토콜을 제공한다. 애플리케이션의 용도에 따라 적당한 프로토콜을 사용한다.
1) HTTP
    - 타입 : 텍스트
    - 장점 : 자주 사용되며 ES 버전간 호환성
    - 단점 : HTTP 오버헤드
2) Native
    - 주로 노드 간의 저수준 통신에 쓰인다. 큰 데이터 블록을 빠르게 가져올 때 유용
    - 타입 : 바이너리
    - 장점 : 빠른 네트워크 계층. 대규모 색인 작업에 최적
    - 단점 : ES 버전간 호환 X
3) Thrift
    - 타입 : 바이너리
    - 장점 : 쓰리프트 플러그인에 따름
    - 단점 : 바이너리

## 환경 설정
- jvm.options
- elasticsearch.yml (실행 호나경 설정)
    - cluster.name: <클러스터명>
    - node.name: <노드명>
    - path.data: [경로]
        - 색인된 데이터를 저장하는 경로를 지정. 디폴트는 엘라스틱서치가 설치된 홈 경로 아래의 data 폴더. 배열 형태로 여러개의 경로값의 입력이 가능하기 때문에 한 서버에서 디스크 여러개를 사용 가능
    - path.logs: <경로>
    - bootstrap.memory_lock: true
        - 엘라스틱서치가 사용중인 힙메모리 영역을 다른 자바 프로그램이 간섭 못하도록 미리 점유하는 설정. 엘라스틱서치는 많은 메모리를 필요로 하기 때문에 항상 true로 놓고 사용하는게 좋다.
    - http.port: <포트 번호>
    - transport.port: <포트 번호>
        - 엘라스틱서치 노드들끼리 서로 통신하기 위한 tcp 포트를 설정. default 9300
    - discovery.seed_hosts: [<호스트1>, <호스트2>, ...]
        - 클러스터 구성을 위해 바인딩 할 원격 노드의 IP 또는 도메인 주소

## 기본 REST API 명령어
````
curl -X GET "localhost:9200/_cat"
curl -X GET "localhost:9200/_cat/indices?v"
curl -X GET "localhost:9200/_all/_search?pretty"

# 기본 CRUD
curl -X GET "localhost:9200/customer/_search?pretty"
curl -X POST "localhost:9200/customer/_doc?pretty" - H "Content-Type: application/json" -d "{\"name\": \"gildong\"}"
curl -X PUT "localhost:9200/customer/_doc/1?pretty" - H "Content-Type: application/json" -d "{\"name\": \"gildong2\"}"
curl -X PUT "localhost:9200/customer/_doc/1?pretty"
curl -X PUT "localhost:9200/customer/_doc/1?pretty&filter_path=_source"
curl -X PUT "localhost:9200/customer/_doc/1?pretty&filter_path=_source.name"
curl -X DELETE "localhost:9200/customer/_doc/1?pretty"
curl -X DELETE "localhost:9200/customer?pretty"
````

## Search API
아래 링크에서 테스트용 데이터를 다운받고 등록한다.  
[Sample Bulk Data](https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json)
````
# 벌크 데이터 등록
curl -X POST "localhost:9200/bank/account/_bulk?pretty&refresh" -H "Content-Type: application/json" --data-binary "@accounts.json"

# 검색
curl -X GET "localhost:9200/bank/account/_search?pretty"
curl -X GET "localhost:9200/bank/account/_search?q=age:39&pretty"
curl -X GET "localhost:9200/bank/account/_search?q=age:<21pretty"
````
Response
- took : 검색하는데 소요된 시간 (단위: ms)
- timed_out : rjatortl tlrks chrhk duqn
- _shards : 검색한 샤드 수 및 검색에 성공 또는 실패한 샤드의 수
- hits : 검색 결과
    - total: 검색 조건과 일치하는 문서의 총 개수
    - max_Score: 검색 조건과 결과 일치 수준의 최댓값
    - hits : 검색 결과에 해당하는 실제 데이터들 (기본값으로 10이 설정되며, size를 통해 조절 가능)
        - _score : 해당 도큐먼트가 지정된 검색 쿼리와 얼마나 일치하는지를 상대적으로 나타내는 숫자 값이며 높을수록 관련성이 높다
    - sort : 결과 정렬 방식을 말하며 기본은 _score 내림차순(desc)이고, 다른 항목일 경우 오름차순(asc)으로 끼본 설정됨 (_score 기준일 경우 노출되지 않음)

### Query DSL
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html

### 질의(Query) & 필터(Filter)
- 질의(Query) : 루씬 내부의 득점(score) 알고리즘을 사용해 일치하는 결과의 점수를 계산하는 것
- 필터(Filter) : 점수를 계산하지 않고 일치하는 것만 찾는것. 일반적으로 빠르며 캐시를 지원한다

### Query Context & Filter Context
- Query Context
    - Query Context에서 사용되는 쿼리절은 "해당 도큐먼트가 쿼리절과 얼마나 잘 일치하는가?"라는 질문에 응답하는데, 도큐먼트가 얼마나 잘 일치하는지를 스코어(_score, 관련성 점수, relevance score)로 표현한다.
- Filter Context
    - Filter Context에서 사용되는 쿼리절은 "해당 도큐먼트가 쿼리절과 일치합니까?"라는 질문에 응답하는데, 그 대답은 true 또는 false이며 점수는 계산하지 않는다.
````
curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}
'
````

## Scroll
RDBMS의 cursor와 같은 기능을 제공한다.  
- search 쿼리에 scroll 값을 추가한다.
- scroll이 생성되면 엘라스틱서치는 지정한 시간만큼 search context를 유지한다.
- 검색 시간을 고려하여 scroll 시간을 정하는게 좋다.
- search context는 지정한 시간이 지나면 자동 소멸되지만, 데이터가 예상한 시간보다 빨리 조회 및 사용될 경우에는 직접 scroll을 삭제해주는게 좋다.
````
curl -X POST "localhost:9200/customer/_search?scroll=1m&size?pretty" -H "Content-Type: application/json"
curl -X POST "localhost:9200/_search?pretty" -H "Content-Type: application/json" -d "{\"scroll\": \"1m\", \"scroll_id\": \"AEFUAESR45SDZF4ASDFAFEEEEE49diASD\"}"
curl -X DELETE "localhost:9200/_search/scroll" -H "Content-Type: application/json" -d "{ \"scroll_id\": \"AEFUAESR45SDZF4ASDFAFEEEEE49diASD\"}"
````

## 스크립트
- 많은 도큐먼트를 처리해야 할 때, 스크립트를 이용해 도큐먼트를 수정하면 네트워크 트래픽이 줄어들고 성능이 향상된다. (스크립트를 사용하지 않으면 도큐먼트를 열고 필드를 변경한 후 돌려줘야 한다.)
- 스크립트는 여러 방법으로 스크립트를 읽을 수 있다.
- 동적 스크립트는 보안상의 이유로 기본값인 '사용 안 함'으로 설정되어 있다. 자바스크립트나 파이썬 같은 동적 스크립트 언어를 사용하려면 '사용'으로 바꿔야 한다. elasticsearch.yml 파일에서 script.disable_dynamic 값을 true로 변경하고 클러스터를 재식해야 한다.
- 가장 안전한 방법은 config/scripts 폴더에 해당 스크립트를 파일로 제공하는 것이다. 엘라스틱서치는 주기적으로(default 60초) 폴더를 스캔한다. 스크립트 언어는 자동으로 파일 확장에 의해 탐지되고 스크립트 이름은 파일 이름에 존재한다.

## 리퍼(river)
외부 원본 데이터에서 데이터를 읽어 클러스터에 저장하는 서비스(MoggonDB, RabbitMQ, Redis, JDBS, 트위치 등)  
리버는 프로토타이핑으로 하기에 좋은 도구이지만 이슈가 발생하면 엘라스틱서치 클러스터가 불안정해질 수 있다. 따라서 분리된 애플리케이션에서 데이터를 수신하는 것이 모범 사례다. Logstash가 그 중 하나이다.
- 장점
    - 클러스터 시작 또는 노드 실패로 다른 노드로 마이그레이션될 때, 엘라스틱서치가 자동 재시작을 관리한다.
    - 플러그인으로 쉽게 배포 가능
- 단점
    - 리버의 실패 또는 오작동으로 인해 노드 또는 클러스터가 멈출 수 있다.
    - 리퍼 실행 시 리버의 부하를 조정할 수 있는 기능이 없어서 일부노드가 높은 오버헤드에 걸려 전체적인 성능을 줄일 수 있다.
    - 리버를 변경하려면 클러스터 재시작을 해야한다.
    - 다중 노드 환경에서 리버를 디버깅하기가 어렵다.

## 클러스터와 노드 모니터링
클러스터 레벨에서 일어날 수 있는 이슈들
- 노드 오버헤드 : 노드가 너무 많은 샤드를 관리하느라 전체 클러스터에 병목이 생김
- 노드 셧다운
- 샤드 재배치 : 특정 샤드의 온라인 상태를 얻을 수 없으면 샤드 재배치 관련 문제나 변질이 발생한다.
- 매우 큰 샤드 : 샤드가 너무 크면 루씬에서 세그먼트 병합이 많이 일어나기 때문에 색인 성능이 떨어진다.
- 비어있는 색인과 샤드 : 모든 샤드에 많은 쓰레드가 동작하기 때문에 데이터가 없는 색인과 샤드는 메모리와 자원을 소비한다. 많은 색인과 샤드를 사용하지 않는다면 클러스터 성능은 떨어진다.

### 클러스터 Health check
````
curl -X GET "localhost:9200/_cluster/health?pretty"
````
status
- green
- yellow : 노드 또는 샤드의 손실을 의미하지만 클러스터 기능에는 손상이 없음을 의미한다. 주로 여러 복제본의 누락을 의미하며 동작 중인 개별 샤드의 복사본인 최소한 하나는 있다. 읽기, 쓰기 기능이 동작하고 있음을 의미한다.
- red : 주요 샤드의 손실을 의미한다. 색인의 상태가 빨간색이라 저장할 수 없어서 결과가 완성된 것이 아닐 수 있으며, 결과를 부분적으로 리턴한다. 다운된 노드를 재시작하고 필요하다면 복제본을 생성해야 한다.

### 노드 통계
````
curl -X GET "localhost:9200/_nodes"
curl -X GET "localhost:9200/_nodes/name1,name2"
curl -X GET "localhost:9200/_nodes/name1,name2/jvm,os,..."
````

## 스냅샷
Repository 설정
````
path.repo: ["/tmp/es/backup"]
````
````
# 생성
curl -X PUT "localhost:9200/_snapshot/backup?pretty" -H "Content-Type: application/json" -d "{ "type": "fs", "settings": { "location": "/tmp/es/backup", "compress": true } }"

# 확인
curl -X GET "localhost:9200/_snapshot/backup?pretty"

# 삭제
curl -X DELETE "localhost:9200/_snapshot/backup?pretty"

# 검증
curl -X POST "localhost:9200/_snapshot/backup/_verify?pretty"
````

스탭샷 만들기
````
curl -X PUT "localhost:9200/_snapshot/backup/snap_01?wait_for_completion=true"
{
    "indices": "accounts-*",
    "ignore_unavailable": true,
    "include_global_state": false,
    "metadata": {
        "taken_by": "kimchy",
        "taken_because": "backup before upgrading"
    }
}
````

스냅샷 조회
````
curl -X GET "localhost:9200/_snapshot/bakcup/_all?pretty"
````

스냅샷 복수
````
curl -X POST "localhost:9200/_snapshot/bakcup/snap_01/_resotre"
{
    "indices": "accounts-*",
    "ignore_unavailable": true,
    "include_global_state": true,
    "rename_pattern": "account-(.+)",
    "rename_replacement": "resotred_account-$1"
}
````

## Java Client
- JVM 언어 기반의 애플리케이션은 엘라스틱서치와 통합할 수 있는 네이티브 프로토콜을 사용할 수 있다.
- 네이티브 프로코톨은 엘라스틱서치와 통신할 수 있는 가장 빠른 프로토콜이다.
- 그 이유는 바이너리 특성, 데이터를 빠르게 변환하는 네이티브 직렬화/역직렬화, 비동기 통신 방식, 홉감소(Rest 호출은 두번의 연결을 해 데이터 노드와 직접 통신하는데, 네이티브 클라이언트는 데이터 노드에 바로 연결할 수 있다.) 같은 요인들 때문이다.
- 단, 새로운 버전이 나올때마다 네이티브 프로토콜이 진화하기 때문에 버전 간 호환성을 보장하지 않는다.

### Elasticsearch with Spring
- 스프링에서 지원하는 [Spring Data Elasticsearch](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#preface) 사용
- 엘라스틱서치에서 지원하는 Rest Client
- 엘라스틱서치에서 지원하는 Tranport Client (deperated)

### Index wndy aoroqustn
- routing : 색인에 사용할 샤드를 제어한다.
- refresh : 도큐먼트를 색인한 후 새로고침을 강제한다. 이 옵션을 이용하면 도큐먼트를 색인한 후에 색인된 도큐먼트를 바로 검색할 수 있다.

## Plugin
다양한 플러그인을 제공한다.
- https://www.elastic.co/guide/en/elasticsearch/plugins/7.7/index.html


## 성능 최적화

### Document 관점
1) Index와 Shard 튜닝
    - 데이터 노드 크기
        - 샤드 크기를 설정하는 가장 쉬운 판단 기준
        - 데이터 노드가 3개라면 기본 샤드 크기를 3개로 설정하고, 5개면 5개로 설정한다.
        ````
        index.number_of_shard: 3
        ````
    - CPU 코어 크기
        - 기본적으로 엘라스틱서치는 병렬처리를 기반으로 동작한다. 따라서 CPU core 크기를 샤드 크기를 정할 때 판단 기준으로 사용할 수 있다.
        - 샤드 크기를 늘리면 일반적인 색인과 검색에 대한 성능 향상 효과를 얻을 수 있지만 적절한 크기 이상으로 설정하면 오히려 성능 저하와 운영상의 어려움을 겪을 수 있으니 주의가 필요하다.
        - azure에서는 (CPU core * node) +/- 1로 할때 가장 큰 퍼포먼스틀 낸다고 가이드한다.
    - Document 크기
        - 샤드 하나의 크기는 50GB 이하로 정의하는 것이 좋다. 이 이상도 가능하지만 문제 발생 시 복구와 검색 성능에 나쁜 영향을 미친다.
        - 다음은 Document 하나의 크기를 기준으로 한 크기 정의이다.
        ````
        # Document 1개 크기 : 100KB
        # Total document count : 10,000,000개
        # Total indexing 크기 : 954GB
        index.number_of_shard: 50
        ````
2) Modeling
    - 주 설정 필드
    - _all
        - 모든 필드의 문자열에 대한 색인 작업 결과를 가지고 검색할 수 있게 지원하는 통합 필드
        - 모든 필드에서 색인 작업을 수행하므로 CPU 사용량이 많고, 인덱스 크기도 늘어난다.
        - Documnent에 대한 풀 텍스트 검색 기능을 사용하는 것이 아니라면 disabled로 설정하여 자원을 절약할 수 있따.
    - _routing
        - 필요에 따라 특정 샤드를 지정하여 사용한다.
    - _id
        - 이 필드를 기준으로 도큐먼트를 어느 샤드에 저장할지 결정된다.
        - 특정 샤드로 데이터가 몰리지 않는지 확인해야 한다.
        지정하지 않으면 해시 문자열로 가지게 되므로 랜덤하게 배치된다.

### Operation 관점
1. Bulk 요청 (refresh interval, request size, timeout 설정)
    - 엘라스틱서치에서 색인 요청을 최적화하는 방법은 멀티스레딩을 이용하여 작은 규모의 색인 요청을 여러번 하는 것이다.
        1) Disable replica & refresh_interval : 복제와 리프레시 설정 off
        2) Document read : 색인할 문서를 읽는다.
        3) Create bulk request : 벌크 색인 요청을 생성한다.
        4) Bulk request : 벌크 색인 요청
        5) Repeat "Step 2" : 이 과정을 "Documnet read"부터 반복
        6) Optimize : 최적화 작업을 수행
        7) Enable replica & refresh_internal : 복제와 리프레시 설정 복구(on)
    - Bulk 요청 크기
        - 1,005,000개 사이의 도큐먼트나 515MB 사이의 물리적인 크기로 정의하는 것이 좋다.
    - Exception 모니터링
        - EsRejectedExecutionException, TOO_MANY_REQEST, NoNodeAvailableException, Timeout exception 등
        - 위 Exception은 요청이 과하거나 서버에서 처리할 수 있는 용량을 넘어선 것이다. scale out을 고려하거나 요청을 줄여야 한다.

## Reference
- [ElasticSearch Cookbook](http://www.yes24.com/Product/Goods/24301881)

