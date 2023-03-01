# Spark 3

#### DataFrame Example: DataFrame and SQL

스파크에서는 사용자가 SQL이나 DataFrame으로 표현한 비즈니스 로직을 실제 코드를 실행하기 전에 기본 실행 계획으로 컴파일한다. 스파크 SQL은 모든 DataFrame을 테이블이나 뷰로 등록한 후 SQL 쿼리를 사용할 수 있도록 한다. createOrReplaceTempView 메서드를 호출하면 모든 DataFrame을 테이블이나 뷰로 사용할 수 있다. SQL 쿼리는 DataFrame 코드와 같은 실행계획으로 컴파일되므로 둘 간의 성능 차이는 없다.

```
// spark in scala
flightData2015.createOrReplaceTempView("flight_data_2015")
```

DataFrame에 spark.sql 메서드를 통해 SQL 쿼리를 수행하면 새로운 DataFrame을 반환한다.

```scala
// spark in scala
// ex1
val sqlWqy = spark.sql("""
SELECT DEST_COUNTRY_NAME, count(1)
FROM flight_data_2015
GROUP BY DEST_COUNTRY_NAME
""")

// ex2
val dataFrameWay = flightData2015
	.groupBy("DEST_COUNTRY_NAME")
    .count()

// 실행계획
sqlWay.explain
dataFrameWay.explain
```

두 가지 실행 계획은 모두 동일한 기본 실행 계획으로 컴파일된다. (\* FileScan, HashAggregate, Exchange hashpartitioning, HashAggregate)

다음 코드 예제는 DataFrame의 특정 칼럼 값을 스캔하면서 이전 최댓값보다 더 큰 값을 찾는 max 함수를 사용한다. max 함수는 필터링을 수행해 단일 로우를 결과로 반환하는 트랜스포메이션이다.

```
spark.sql("SELECT max(count) from flight_data_2015").take(1)

import org.apache.spark.sql.functions.max
flightData2015.select(max("count")).take(1)
```

스파크는 기본 요소인 저수준 API와 구조적 API 그리고 추가 기능을 제공하는 일련의 표준 라이브러리로 구성되어 있다. 스파크의 라이브러리는 그래프 분석, 머신러닝, 스트리밍 등 다양한 작업을 지원하며, 컴퓨팅 및 스토리지 시스템과의 통합을 돕는 역할을 한다.

#### 운영용 애플리케이션 실행하기

spark-submit 명령어를 통해 대화형 shell에서 개발한 프로그램을 운영용애플리케이션으로 전환할 수 있다. 애플리케이션 코드를 클러스터에 전송해 실행시키는 역할을 하는데, 제출된 애플리케이션은 작업이 종료되거나 에러가 발생할 때까지 실행된다. 스파크 애플리케이션은 stand-alone, Mesos, YARN 클러스터 매니저를 이용해 실행된다. spark-submit 명령에 애플리케이션 실행에 필요한 자원과 실행 방식, 다양한 옵션을 지정할 수 있다. 사용자는 스파크가 지원하는 프로그래밍 언어(scala 등)으로 애플리케이션을 개발한 다음 실행할 수 있다.

```
// spark in scala
./bin/spark-submit \\
--class org.apache.spark.examples.SparkPi \\
-- master local \\
./example/jars/spark-examples_2.11-2.2.0.jar 10
```

로컬 머신에서 실행되도록 설정되었으며, 실행에 필요한 JAR 파일과 관련 인수도 함께 지정했다. master 옵션의 인수를 변겨아면 스파크가 지원하는 spark-standalone, Mesos, YARN 클러스터 매니저에서 동일한 애플리케이션을 실행할 수 있다.

#### Dataset: 타입 안정성을 제공하는 구조적 API

Dataset은 자바와 스칼라의 정적 데이터 타입에 맞는 코드, 즉 정적 타입 코드를 지원하기 위해 고안된 스파크의 구조적 API이다. 타입 안정성을 지원하며, 동적 타입 언어인 파이썬과 R에서는 사용할 수 없다.DataFrame은 다양한 데이터 타입의 테이블형 데이터를 보관할 수 있는 Row 타입의 객체로 구성된 분산 컬렉션이다. Dataset API는 DataFrame의 레코드를 사용자가 자바나 스칼라로 정의한 클래스에 할당하고 자바의 ArrayList, 스칼라의 Seq 객체 등의 고정 타입형 컬렉션으로 다룰 수 있는 기능을 제공한다. Dataset API는 타입 안정성을 지원하므로 초기화에 사용한 클래스 대신 다른 클래스를 사용해 접근할 수 없다. 이로 인해 다수의 소프트웨어 엔지니어가 잘 정의된 인터페이스로 상호작용하는 대규모 애플리케이션을 개발하는 데 유용하다.

#### 구조적 스트리밍

구조적 스트리밍은 Spark 2.2 버전에서 안정화된 스트림 처리용 고수준 API이다. 구조적 스트리밍을 사용하면 구조적 API로 개발된 배치 모드의 연산을 스트리밍 방식으로 실행할 수 있으며, 지연 시간을 줄이고 증분 처리할 수 있다. 배치 처리용 코드를 일부 수정하여 스트리밍 처리를 수행하고 값을 빠르게 얻을 수 있다는 장점이 있다. 또한 프로토타입을 배치 잡으로 개발한 다음 스트리밍 잡으로 변환할 수 있으므로 개념 잡기가 수월하며, 앞선 모든 작업이 데이터를 증분 처리하면서 수행된다.

#### 머신러닝과 고급 분석

스파크에 내장된 머신러닝 알고리즘 라이브러리인 MLlib를 사용해 대규모 머신러닝을 수행할 수 있다. MLlib을 사용하면 대용량 데이터를 대상으로 preprocessing, munging, model training, prediction을 할 수 있다. 또한 구조적 스트리밍에서 예측하고자 할 때도 MLlib에서 학습시킨 다양한 예측 모델을 사용할 수 있다. 스파크는 classification, regression, custering, deep learning에 이르기까지 머신러닝과 관련된 정교한 API를 제공한다.

데이터 전처리에 사용하는 다양한 메서드를 제공하며, MLlib의 트랜스포메이션 API(TrainCalidationSplit이나 CrossValidator)를 사용해 학습 데이터셋과 테스트 데이터셋을 생성할 수 있다. 일반적인 트랜스포메이션을 자동화하는 다양한 트랜스포메이션도 제공한다. 그중 하나가 StringIndexerer이다.

#### 저수준 API

스파크는 RDD를 통해 자바와 파이썬 객체를 다루는 데 필요한 다양한 기본 기능(저수준 API)을 제공한다. 스파크의 거의 모든 기능은 RDD를 기반으로 만들어졌다. DataFrame 연산도 RDD를 기반으로 만들어졌으며 편리하고 효율적인 분산 처리를 위해 저수준 명령으로 컴파일된다. 원시 데이터를 읽거나 다루는 용도로 RDD를 사용할 수 있지만 대부분은 구조적 API를 사용하는 것이 좋다. 하지만 RDD를 이용해 파티션과 같은 물리적 실행 특성을 결정할 수 있으므로 DataFrame보다 더 세밀한 제어를 할 수 있다.드라이버 시스템의 메모리에 저장된 원시 데이터를 병렬처리하는 데 RDD를 사용할 수 있다. RDD는 스칼라뿐만 아니라 파이썬에서도 사용할 수 있다. 하지만 두 언어의 RDD가 동일하진 않다. 언어와 관계없이 동일한 실행 특성을 제공하는 DataFrame API와는 다르게 RDD는 세부 구현 방식에서 차이를 보인다. 최신 버전의 스파크에서는 기본적으로 RDD를 사용하지 않지만, 비정형 데이터나 정제되지 않은 원시 데이터를 처리해야 한다면 RDD를 사용해야 한다.
