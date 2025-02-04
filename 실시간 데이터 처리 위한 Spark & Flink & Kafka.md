---
UP:
  - "[[1️⃣ Figurehead]]"
  - "[[2️⃣ Engineering]]"
github: https://github.com/noggong/realtime_streaming
---
# Data pipeline
- Data pipeline 에 의해 만들어진 데이터는 BI (Business Inteligence), 데이터 분석, ML, 검색 색인 구축 등 다양한 목적으로 사용됨
## 사용 예시 
- 웹 / 앱의 사용자 로그 정보 수집 및 정제
- 공장 내 IOT 장비의 센서들로부터 데이터 수집 및 정제
- 실시간 주가 정보 수집 ^e82487
- 게임, etc
## ETL
### 추출 (Extract)
- 다양한 데이터 소스들로부터 input 데이터 추출
### 변환 (Transformation)
- 추출된 데이터를 정체 및 변환
### Load
- 변환된 데이터를 다른 데이터 저장소로 적재

# 데이터 처리 관련 주요 아키텍쳐
# Case 웹 사이트 내의 각 URL 별,  현재 시점의 누적 조회 수를 계산
- Input : URL, 현재 시점 /  output : 조회수
- ## 증분 방식
	- RDBMS 를 사용
	- 각 방뭄자의 id 별로 특정 url 방문시 view_count +1 
	- 단점
		- 트래픽 증가 문제 : 확장성 면에서는 좋지 않음
		-  update query 가 빈번하게 발생 
		- rows 증가 문제
	- 단점 해결 방식
		- 트래픽 증가 문제
			- 큐를 추가하여 이벤트를 모아 일괄 업데이트 수행
		- RDBMS 확장 필요
			- 수직 확장 (Scale up)
			- 수평 확장 (샤딩)
- ## Lambda architecture
	- 도메인과 무관 하게 빅데이터 파이프라인에서 널리 쓰이는 아키텍쳐
	- 크게 3가지 레이어로 구성
		- ### 1. Batch Layer
			- 주기적 (하루에 한,두번)으로 미리 계산된 view 를 생성
			- master dataset 관리
			- ==하루 2~3번 Group By 연란 미리 수행 → 부하감소==
			- 분산 시스템에 더 맞는 저장소 사용 → 확장성, 내결함성 확보 및 수평 확장 기능 (hdfs, 카산드라)
			- client 가 매 질의 할때마다 연산 x
			- 오류가 발생하더라도 새로운 Batch View 로 대체 가능
		- ### 2. Speed Layer
			-  ==Batch Layer 에서 커버하지 못하는 최신 데이터 view 를 생성==
			- 오류가 발생해도 빠른 시간 내에 새로운 view 가 생성되어 대체됨
			- 전체 시스템의 복잡성을 담당
		- ### 3. Serving Layer
			- client 가 Batch Layer, Speed Layer 에서 만들어진 view 를 쿼리 가능하게 하는 역할
	- ![[스크린샷 2025-01-06 오후 9.05.15.png]]
	- ## 단점
		- Batch Layer, Speed Layer 등 동시에 운영해야 하기때문에 운영 복잡성이 커짐
	- 대안
		- Batch Layer 대신 실시간 처리로 대체 한다 (Kappa  architecture / Delta architecture)
# 실시간 데이터 처리 (스트리밍) 란
## 실시간 기준

^a475b4

- 일반적인 웹 / 네트워크에서는 Soft / Near realtime 을 대상으로 함

| 분류   | 예시                   | 지연정도                | 허용 가능 지연     |
| ---- | -------------------- | ------------------- | ------------ |
| Hard | 심박 조율기, ABS 브레이크     | 수 micro ~ 수 milli 초 | 허용 X         |
| Soft | 항공편 예약, 온라인 주식 시세    | 수 mill ~ 수초         | 어느 정도 허용 가능 |
| Near | 화상 통화, 로그데이터 분석 대시보드 | 수 초 ~ 수 분           | 허용 가능        |
## 스트리밍 데이터 아키텍쳐
수집 → 메시지 큐 → 분석 → 저장 → 데이터 접근
ex) 트위터
### 수집
- 사용자가 트위터에 글을 쓰면, 트위터 서버는 데이터를 수집
### 메시지 큐 
- 전 세계의 사용자들르부터 수집된 데이터를 각 데이터 센터의 메시지 큐에 저장 (대표 : kafka )
### 분석
- 게시된 글을 분석 (ex: 게시글의 팔로워들을 추출, 이상한 내용이 포함됐는지, AI 가 쓴 글인이 확인등) (대표: Flink, spark)
### 저장
- 분석 결과를 클라이언트가 빠르게 가져갈수 있도록, in-memory  DB 에 저장
### 접근
- 다수의 트위터 클라이언트들을 트위터 서버에 연결
# Apache Spark 
- 데이터 센터나 클라우드에서 대규모 분산 데이터 처리를 하기 위해 설계된 통합형 엔진
## spark 주요 설계 철학
- ### 1. 속도
	- 디스크 I/O 를 주로 사용하는 하둡 맵리듀스 처리 엔진과 달리, ==중간 결과를 메모리에 유지== 하기 때문에 훨씬 더 빠른 속도로 같은 작업을 수행 가능
	- ==질의 연산을 방향성 비순환 그래프 (DAG - Directed acyclic graph)== 로 구성
		- DAG 의 스케쥴러와 질의 최적화 모듈은, 효율적인 연산 그래프를 만들고 각각의 태스크로 분해하여 클러스터의 워커 노드위에서 병렬 실행 될수 있도록 함
	- 물리적 실행 엔진인 텅스텐은 전체 코드 생성 기법을 사용해, 실행을 위한 간결한 코드를 생성
- 2. 사용 편리성 
	- Dataframe, Dataset 과 같은 고수준 데이터 추상화 계층 아래에 RDD(Resiient Distributed dataset) 라 불리는 단순한 자료구조를 구축해 단순성을 실현함
	- 연산의 종류로 transformation / action 의 집합과 단순한 프로그래밍 모델을 제공
	- 여러 언어 제공
- ### 3. 모듈성
	- Spark 에 내장된 다양한 컴포넌트들을 사용해 다양한 타입의 워크로드에 적용이 가능함
	- ==특정 워크로드를 처리하기 위해 하나의 통합된 처리 엔진을 가짐==
		- 배치, 스트리밍, SQL 질의 등을 외부 엔진이 아닌 자체에서 모두 가능함
- ### 4. 확장성
	- 저장과 연산을 모두 포함하는 하둡과는 달리, 빠른 병렬 연산에 초점
	- 수많은 데이터 소스 (하둡, 카산드라, 몽고, s3..등)로 부터 데이터를 읽어 들일 수 있음.
	- 여러 파일 포햇 (txt, csv, parquet, roc, hdfs..등) 과 호환 가능
	- 이 외에 많은 서드파티 패키지 목록 사용 가능

## Spark 애플리케이션 구성 요소
![[스크린샷 2025-01-06 오후 9.50.31.png]]
- ### Cluster manager : 클러스터 매니저
	- 애플리케이션의 리소스 관리
		- 드라이버가 요청한 실행기 프로세스 시작
		- 실행 중인 프로세스를 중지하거나 재시작
		- 실행자 (excutor) 프로세스가 사용할 수 있는 최대 CPU 코어 개수 제한등
	- 종류
		- Standalone : 보통 로컬에서 
		- Apache Mesos : 일반적으로 많이 사용
		- Hadoop Yarn : 일반적으로 많이 사용
		- Kubernetes
- ### Driver: 드라이버
	- 스파크 애플리케이션의 실행을 관장하고 모니터링
		- 클러스터 매니저에 메모리 및 CPU 리소스를 요청
		- 애플리케이션 로직을 스테이지와 태스크로 분할
		- 여러 실행자에 태스트를 전달
		- 태스크 실행 결과 수집
	- 1개의 스파크 애플리케이션에는 1개의 드라이버만 존재
	- 드라이버 프로세스가 어디에 있는지에 따라, 스파크에는 크게 두가지 모드가 존재
		- 클러스터 모드 - 드라이버가 클러스터 내의 특정 노드에 존재 (일반적으로 많이 사용)
		- 클라이언트 모드 - 드라이버가 클러스터 외부에 존재
- ### Excutor : 실행기
	- 스파크 드라이버가 요청한 태스크들을 받아서 실행하고 그 결과를 드라이버르 변환
		- JVM 프로세스
		- 각 프로세스는 드라이버가 요청한 태스크들을 여러 태스크 슬롯 (스레드) 에서 병렬로 실행
		- 실행기 → 프로세스 / task slot → 쓰레드
- ### Spark Session: 스파크 세션
	- 스파크 코어 기능들과 상호 작용할 수 있는 진입점 제공, 그 API 로 프로그래밍을 할 수 있게 해주는 객체
		- spark-shell 에서는 기본적으로 제공
		- 스파크 애플리케이션에서는 사용자가 SparkSession 객체를 생성해 사용해야함
- ### Job: 잡
	- 스파크 액션 (ex: save(), collect()) 에 대한 응답으로 생성되는 여러 태스크로 이루어진 병렬 연산![[스크린샷 2025-01-06 오후 9.59.17.png]]
- ### Stage: 스테이지
	- 스파크 각 Job 은 스테이지라 불리는 서로 의존성을 가지는 다수의 태스크 모음으로 나뉨![[스크린샷 2025-01-06 오후 9.59.51.png]]
- ### Task : 태스크
	- 스파크 각 잡별 실행기로 보내지는 작업 할당의 가장 기본적인 단위
	- 개별 task slot 에 할당 되고 데이터의 개별 파티션을 가지고 작업 ! ![[스크린샷 2025-01-06 오후 10.00.48.png]]

## 스파크 연산의 종류
### 트랜스포메이션 (Transformatiopn)
- Immutable (불변) 인 원본 데이터를 수정하지 않고, 하나의 RDD 나 Dataframe 을 새로운 RDD 나 Datafarme 으로 변형
- map(), filter(), flatMap(), select(), groupby(), orderby() 등
- #### Narrow Transformation
	- input: 1개의 파티션
	- output : 1 개의 파티션
	- 파티션 간의 데이터 교환이 발생하지 않음.
	- filter(), map(), coalesce()![[스크린샷 2025-01-06 오후 10.05.30.png]]
- #### Wide Transformation
	- 연산지 파티션끼리 데이터 교환이 발생
	- groupby(), orderby(), sortByKey(), reduceByKey()
	- 단 join 의 경우 두 부모 RDD/Dataframe 이 어떻게 파티셔닝 되어 있냐에 따라 narror 일수도 wide 일수도 있음.![[스크린샷 2025-01-06 오후 10.06.45.png]]
### 액션 (Action)
- Immutable (불변) 인 인풋에 대해, Side effect (부수 효과)를 포함하고, 아웃풋이 RDD 혹은 Dataframe 이 아닌 연산
	- count() : ouput int, 
	- collect() : output array
	- save() : ouput void
### Lazy evaluation (지연 평가)
- 모든 transformation 은 즉시 평가 (연산) 되지 않고, ==계보 (lineage) 라 불리는 형태로 기록==
- ==transformation 이 실제 계산되는 시점은 action 이 실행되는 시점==
- action 이 실행될때, 그 전까지 기록된 모든 transformation 들의 지연 연산이 수행된
- #### 장점
	- 스파크가 연산 쿼리를 분석하고, 어디를 최적화 할지 파악하여, ==실행 계획 최적화==가 가능
		-  eager evaluation 이라면, 즉시 연산이 수행되기 때문에 최적화의 여지가 없다.)
	- ==장애에 대한 데이터 내구성을 제공==
		- 장애 발생 시, 스파크는 기록된 lineage 를 재실행 하는 것만으로 원래 상태를 재생성 할 수 있음
# 배치 프로세싱 (Spark SQL, RDD, 
- ## 1. **RDD와 SQL(DataFrame/Dataset)의 차이**

| 특징         | RDD               | SQL(DataFrame/Dataset)             |
| ---------- | ----------------- | ---------------------------------- |
| **API 레벨** | 저수준 API           | 고수준 API                            |
| **최적화**    | 사용자가 직접 최적화       | Catalyst Optimizer와 Tungsten 엔진 사용 |
| **형식 지원**  | Java/Scala 객체로 저장 | 구조화된 데이터를 컬럼 형식으로 저장               |
| **성능**     | 상대적으로 느림          | 최적화 덕분에 빠름                         |
| **사용 용이성** | 코드가 복잡            | 간결하고 SQL-like 문법 지원                |

---

- ## 2. **SQL(DataFrame/Dataset)이 더 효율적인 이유**
    1. **Catalyst Optimizer**:
        - DataFrame/Dataset은 Catalyst Optimizer를 사용하여 **쿼리를 자동으로 최적화**합니다.
        - RDD는 사용자가 최적화해야 하므로, 복잡한 작업에서는 비효율적일 수 있습니다.
    2. **Tungsten Execution Engine**:
        - SQL 형식은 Tungsten 엔진을 통해 **바이트코드 생성 및 메모리 최적화**를 수행합니다.
        - 이로 인해 Spark는 더 낮은 수준에서 효율적으로 실행됩니다.
    3. **데이터 직렬화**:
        - DataFrame은 컬럼 기반 형식을 사용하여 데이터 직렬화/역직렬화 성능이 뛰어납니다.
        - RDD는 객체 직렬화를 사용하는 경우 성능이 떨어질 수 있습니다.
    4. **메모리 관리**:
        - DataFrame/Dataset은 메모리 관리 및 캐싱을 효율적으로 처리합니다.
        - RDD는 사용자가 직접 메모리 및 파티셔닝 전략을 설계해야 하는 경우가 많습니다.
    5. **쉬운 쿼리 작성**:
        - SQL-like 쿼리로 복잡한 데이터를 쉽게 처리할 수 있습니다.
        - RDD는 함수를 체이닝하고 변환 과정을 직접 구현해야 하므로 상대적으로 코드가 복잡해집니다.

- ## 3. **RDD를 선호하는 경우**
    1. **저수준 데이터 처리**:
        - 복잡한 데이터 변환이 필요하고 SQL 또는 DataFrame API로는 구현하기 어려운 경우.
    2. **비구조화된 데이터**:
        - 데이터가 비구조적이거나 사전 정의된 스키마가 없는 경우.
    3. **사용자 정의 로직**:
        - 특정한 사용자 정의 로직(예: 복잡한 순서 의존 로직)이 필요한 경우.
    4. **최대 유연성 필요**:
        - DataFrame/Dataset은 컬럼 기반으로 동작하기 때문에, RDD가 더 유연한 작업을 지원하는 경우가 있습니다.
- ## 4. **성능 비교**

| 작업 유형              | 더 효율적인 형식                 |
| ------------------ | ------------------------- |
| 데이터 읽기/쓰기 및 간단한 변환 | DataFrame/Dataset         |
| 복잡한 집계 및 SQL 쿼리    | DataFrame/Dataset         |
| 고급 사용자 정의 연산       | RDD                       |
| 머신러닝 및 그래프 처리      | DataFrame/Dataset (MLlib) |
- ## 5. **결론**
    - **DataFrame/Dataset**:
        
        - 대부분의 경우 더 빠르고 효율적이며 최적화를 자동으로 처리합니다.
        - SQL-like 작업이나 구조화된 데이터를 다룰 때 적합합니다.
    - **RDD**:
        
        - 유연성이 필요하거나 저수준 작업을 수행할 때 사용됩니다.
        - 그러나 성능과 최적화 측면에서 DataFrame/Dataset보다 비효율적일 수 있습니다.
> 가능하다면 **DataFrame/Dataset을 기본으로 사용**하고, RDD는 특정 상황에서만 사용하는 것이 추천됩니다.Datafarme)
## RDD (Resilient Distributed Dataset)
- 스파크의 기본 추상화 객체
- ### 주요 구성 요소
	- 의존성 정보
		- 어떤 입력을 필요로 하고 현재의 RDD 가 어떻게 만들어지는지 스파크에게 가르쳐줌
		- 새로운 결과를 만들어야 하는 경우, 스파크는 이 의존성 정보를 참고하고 연산을 재반복해 RDD 를 재생성 할 수 있음
	- 파티션 (지역성 정보 포함)
		- 스카크에서 작업을 나눠 실행기들에 분산해 파티션별로 병렬 연산 할 수 있는 능력을 제공
	- 연산 함수 : Partition => Iterator[T]
		- RDD 에 저장되는 데이터를 Iteratorp[T] (반복자) 형태로 변환
## 파이프라인 만들기
[Page not found · GitHub · GitHub](https://github.com/noggong/realtime_streaming/blob/main/part02/ch02/log_rdd_ex.py)

## Spark UI 접속하기
- pyspark 사용한 파일에서 무한루프로 프로세스 끝내지 않게 한뒤 localhost:4040 접속한다.
```
while True:  
    pass
```

## Join
- 두 RDD 간의 JOIN 이 가능하다
```python 
inner = user_names_rdd.join(user_visits_rdd).sortByKey()  
print(f"inner => {inner.collect()}")  
  
left_outer = user_names_rdd.leftOuterJoin(user_visits_rdd).sortByKey()  
print(f"left => {left_outer.collect()}")  
  
right_outer = user_names_rdd.rightOuterJoin(user_visits_rdd).sortByKey()  
print(f"right => {right_outer.collect()}")  
  
full_outer = user_names_rdd.fullOuterJoin(user_visits_rdd).sortByKey()  
print(f"full outer => {full_outer.collect()}")
```






# Flink
## Abstraction
### Stream 이란?
- 실시간 혹은 거의 준실시간으로 생성되는 데이터의 연속적인 흐름을 의미
- 데이터가 도착하는 대로 처리하여 즉각적인 분석과 의사결정을 가능하게 해야한다.
### Stream Data 에서 해결해야 할 지점 (배치등의 처리와 차이점)
- #### Out-of-order
	- 순서가 바뀐 이벤트 처리를 할 수 있어야 한다.
- #### Late events
	- 늦게 도착한 이벤트를 잘 처리 할수 있어야 한다.
	- Flink : 이벤트 타임 기반 처리, 워터마크 등의 기능을 갖추고 있다.
- #### StateFul processing
	- 윈도우잉, 어그레이션, 패턴매칭 등과 같은 작억을 수행하기 위해, 상태 (state), 즉 이전 이벤트에 대해서 컨텍스트 정보를 유지 하고 있다
	- 정확한 결과를 제공하는데 필수적이다.
- #### Real-time processing
- #### Fault tolerance
- #### Scalability
- #### Integration with other Systems
### 다른 Streaming Tool 과 비교
![[스크린샷 2025-01-13 오후 9.02.37.png]]
==파란색 : 긍정 /  주황 : 부정 / 노랑 : 대체로 긍정 이지만 다른 Tool 에 비해서 약함 ==
- spark stream 은 정해진 크기의 스트림데이터를 미니 배치 느낌으로 처리한다. (bounded stream)
- #### Spark Stream 의 장점
	- 배치 처리와 동일한 API
	- Spark Ecosystem 과의 통합
	- Kafka 와 긴밀하기 통합되어 있어 데이터 파이프라인에서 이미 Kafka 를 사용한 조직이라면 좋은 선택
- #### Kafka Streams 의 장점
	- Lighweight & Simple API
	- Kafaka 와의 강력한 통합
- 요구 사항이 ==Out of Order / Late Events 에 대한 보다 정확한 처리와, 과거의 정보인 State 관리등을 꼭 해야 되는 상황이 아니라면 Spark Streams, kafka Streams 가 매력석인 선택==일수 있다.
- Operator : 스트림 파이프라인을 구성하는 기본 구조, 빌딩 블록
### Flink Stream
![[스크린샷 2025-01-23 오전 10.18.40 1.png]]
- 스트림 데이터에서 해결하던 지즘들에 대해서 정조준 하고 그를 해결하기 위해 토대위에 개발되었다
- unbounded Stream : 스트림은 데이터가 실시간으로 들어오기때문에 스트림 데이터의 크기가 일정하지 않다.  ![[스크린샷 2025-01-13 오후 9.07.35.png]]
- 처리 방식에서 데이터가 도착하는 즉시 처리하는 방식인 Real-Time Steram proceeing 으로 처리한다.
### 고수준 저수준 API
![[스크린샷 2025-01-13 오후 9.20.01.png]]
> Timely : 하다는 것은 각 이벤트의 시간 및 분산된 각 노드가 처리하는 시간에 대한 정보를 기반으로 데이터를 처리해줄 수 있게 해준다는 뜻.

- Flink 에서는 DataStream API / DataSet API 를 주로 사용하는 API
	- 일반적으로 Core API 라고 불려진다.
	- DataStream API : Stream Data 를 대상
	- DataSet API : Batch Data를 대상
- 실시간 데이터 처리에 있어 가장 낮은 추상화된 State Stream Processing 이 필요하진 않다
- 계층 구조를 이루고 있으면, 이전단계가 이후단계를 구성하고 있다.
- DSL / SQL 만을 사용할 경우 Spark/Kafka Streams 를 사용하는게 좋은 선택일수 있음

#### High Level API
- streams 나 batches 데이터를 다루기 위한 추상 데이터 타입 제공
	- DataStreams 및 Datasets
- 대부분의 사용 사례에 적합하여, 사용하기 쉬운 인터페이스 제공
	- map, flatMap, filter, window, reduce, join, coGroup
#### Low Level API
- Stream processing 에 대한 더 많은 제어 제공
	- stateful 및 Timely stream processing
#### 고수준 / 저수준 API 를 통해 2배 곱하는 프로세스 개발
##### High Level API - map()
```java

DataStream<Integer> result = input.map(i -> i * 2)

@Public
@FunctionalInterface
public interface MapFunction<T, O> extends Function, Serializable {
	O map(T value) throws Exception;
}
```
- @FunctionalInterface
	- [[SAM]] (Single Absent Method) : 즉 추천 메소드가 단하나만 존재하는 인터페이스 이므로 Java의 Lambda 식 이나 참조로 해당 오퍼레이터의 계산을 표현할 수 있다는 뜻.

##### Low Level API - ProcessFunction{}
```java
public class MapProcessFunction extends ProcessFunction<Integer, Integer> {
	@Override
	public void processElement(Integer value, Context ctx, Collector<Integer> out) throws Exception {
		out.collect(value * 2); // Simple map operation - doubling the input value
	}

	DataStream<Integer> result = input.process(new MapProcessFunction())
}
```
- Process Function 을 확장한 클래스인 MapProcessFunction 을 통해 스트림 데이터의 각 원소를 두배로 하는 코드
- 위의 예제인 Map 이외에도 모든 DataStream API 를 Stateful Stream Processing 으로 구현할 수가 있다.
- reduce 와 같이 이전 값을 참조 해야한다면 stateful 의 특징인, Context ctx 값을 이용하여 구현하여야 한다.

### Stateful 스트리밍 처리
- 크게 3가지의 state 가 있다
- #### Keyed State![[스크린샷 2025-01-13 오후 10.05.13.png]]
	- 일반적으로 Flink 에서 State 는 Keyed State 를 의미한다.
	- Stream Data 에 state 관리를 위해 키를 생성해준다. 키가 생성된 스트림을 keyed Stream 이라고 불린다.
	- 키에 의해 논리적으로 파티셔닝 
	- 같은 키를 가진, 같은 그룹으로 할당된 레크더들에게만 접근 할수 있다.
- #### Operator State![[스크린샷 2025-01-13 오후 9.54.38.png]]
	- 잘 사용 되지 않음
	- Operator 별로 State 를 관리 (State on a per-operator basis)
	- ![[스크린샷 2025-01-13 오후 9.56.33.png]]
	- ##### Broadcast State (Operator Stator 의 특수한 타입)
### Timely 스트리밍 처리
- 스트림데이터는 시간이 결정되어있지 않다 (non-deterministic)
- 하지만 분석하는 시점에는 시간이 결정되어 있어야 한다.
- 시간을 결정할때 기준이 있어야 한다
	- #### Event Time
	- #### Processing Time
	- #### Ingestion Time (1.12 버전 이후 deprecated)
- #### Timely
	- Timely Stream Processing
		- 시간을 활용해 스트림 데이터를 처리하는 방법을 Timely Stream Processing
		- 알맞은 시간에 스트림데이터를 처리한다는 의미 
	- Processing time  (Wall clock)
		- 처리되는 시간 (process time 아님)
		- 이벤트가 처리되는 ==시점==
	- Event Tiime
		- 이벤트 데이터가 발생한 시점
		- 이벤트 타임을 기반으로 스트림즈 데이터를 처리한다고 했을때 프로세싱타임처럼 기준이 필요하다
		- Internal Event Time Clock  : 각 노드가 가지고 있는 이벤트 타임 정보
			- 워터마크라는 하나의 오브젝트를 통해 인터널 이벤트 타임 클럭의 값을 갱신
	- 
![[스크린샷 2025-01-13 오후 10.23.15.png]]
VS
![[스크린샷 2025-01-13 오후 10.23.35.png]]- 데이터가 순서대로 들어오지 않더라도 워터마크 (W) 에의해 기준이 생기고, 워터마크 기준 좌측 데이터는 우측의 데이터보다 작을수 없다. 이를 어기면 스트림 처리를 하지 않는다
![[스크린샷 2025-01-13 오후 10.26.00.png]]
- watermark 는 소스에서 생성
-  워터마크에 의해 확인된 지연 데이터에 대한 처리를 할수 있다. 지연 허용 시간 또한 설정할수 있다.
- ![[스크린샷 2025-01-13 오후 10.29.24.png]]
- 1분의 지연을 허용한다면 10시5분에 데이터가 타임스탭프로 찍여 들어왔을때, 워터마크는 10시4분으로 찍어서, 1분의 지연을 허용한다. (맞게 이해한건가..)
