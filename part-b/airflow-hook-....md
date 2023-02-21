# Airflow 스케쥴링, Hook, ...

## DAG 스케줄링

### DAG의 스케줄 시작 및 종료

python의 datetime library를 통해 인자로 넘겨주면 매우 같이 간단하게 Airflow의 시작 시간과 종료 시간을 설정할 수 있습니다.

```python
import datetime 
...

dag = DAG(
	dag_id = "YBIGTA_DAG" <- DAG의 Id를 입력합니다.
	start_date = datetime.datetime(2023, 1, 1) <- 시작 날짜 및 시간
	end_date =  datetime.datetime(2023, 1, 5) <- 종료 날짜 및 시간
	...
)
```

참고로 시작 시간은 반드시 설정해주어야 하지만 종료 시간은 필수 인자가 아닙니다. 만약 end\_date를 입력하지 않으면 DAG를 끄기 전까지 계속해서 실행됩니다.

### DAG 스케줄 간격 지정하기

다음과 같이 매크로 형태로 매우 편리하게 DAG을 원하는 간격으로 스케줄링 할 수 있습니다.

```python
dag = DAG(
	dag_id = "YBIGTA_DAG"
	start_date = datetime.datetime(2023, 1, 1)
	schedule_interval = "@daily"
	...
)
```

자주 사용하는 option 을 정리하면 아래와 같습니다.

| 이름       | 의미                 |
| -------- | ------------------ |
| @once    | 1회 스케줄             |
| @hourly  | 매시간 1회 실행          |
| @daily   | 매일 자정에 1회 실행       |
| @weekly  | 매주 일요일 자정에 1회 실행   |
| @monthly | 매월 1월 자정에 1회 실행    |
| @yearly  | 매년 1월 1일 자정에 1회 실행 |

이 외에도 cron 표현식(스케줄러의 정규 표현식)을 사용하여 스케줄링이 가능합니다.

## Airflow Hook

Airflow의 매우 큰 장점 중 하나는 다른 기능들과 연결하기 쉽다는 것입니다. 여기서 말하는 다른 기능들이란 단순히 다른 곳에서 구현된 파이썬 기능을 의미하는 것이 아닌 다른 시스템을 의미합니다. 대표적으로 클라우드나 DB 등이 있습니다.

### Hook

Airflow Hook은 클라우드나 DB와 같은 외부 시스템과 연결하기 위해 Airflow에서 제공하는 인터페이스입니다. Hook없이도 외부 시스템과 연결을 충분히 할 수 있지만 우리는 Hook을 통해서 조금 더 그 과정을 편리하게 할 수 있습니다.

아래는 Airflow를 사용하여 S3에서 데이터를 가져오는 기능을 Hook을 사용하여 구현한 예시입니다.

```python
def _download_from_s3(key, bucket_name, local_path) : 
	hook = S3Hook("AWS_ID") 
	download_file = hook.download_file(key = key, 
		bucket_name = bucket_name, local_path = local_path)
	return download_file

with DAG(
	...
) as dag:

	get_download_file_from_s3 = PythonOperator(
		task_id = "download_from_s3"
		python_callable = _download_from_s3
		op_kwargs = {
            'key': ...,
            'bucket_name': ...,
            'local_path': ...
        } -> python_callable에서 사용하는 함수의 인자들
	)
```

먼저 \_download\_from\_s3라는 method 안에서 S3Hook을 통해 데이터를 가져옵니다. 보시는 것과 같이 몇 가지 인자들을 넣어주는 것으로 데이터를 쉽게 가져오는 것을 확인할 수 있습니다. (AWS\_ID와 관련된 것은 webserver에서 따로 입력해주어야 합니다. 이 부분에 대한 설명을 생략하도록 하겠습니다.) 그리고 이렇게 정의한 method 를 PythonOperator를 통해 불러오면 매우 쉽게 구현할 수 있습니다.

Airflow에는 정말 다양한 Hook이 구현되어 있습니다. 그러나 다른 기능을 사용하는 만큼 연결하고 자하는 기능에 대한 이해도가 없이는 구현이 힘듭니다. 바로 이 부분이 Airflow를 어렵게 하는 부분이기도 합니다. 그러나 Hook을 사용하는 것이 익숙해진다면 Airflow의 편리함에 대해서 몸소 체감하시게 될 것입니다.

## Airflow context

Airflow Context는 Airflow의 Task의 런타임 환경을 나타냅니다. 즉, Context에는 현재 작업 중인 Task의 실행 날짜, Task의 매개 변수, Task의 상태, Task가 속해 있는 DAG 및 특정 Task의 Operator에 대한 참조와 같은 정보가 들어있습니다. 또한, Task는 Context를 사용하여 Task 간에 데이터를 저장 및 검색하고 현재 Task와 DAG에 대한 정보에 액세스할 수 있습니다.

요약하면 Airflow Context는 Task가 런타임 환경과 상호 작용하고, Task 간에 데이터를 공유하고, 현재 Task와 DAG에 대한 정보에 액세스하는 데 사용할 수 있는 정보와 리소스를 포함하는 환경입니다.

## XCom

Airflow Task들은 서로 다른 Task에서 실행되는 분산 시스템으로 설계되었기 때문에 기본적으로 작업 간에 데이터를 공유하지 않습니다. 작업간에 데이터를 공유하려면 네트워크를 통해 데이터를 전송해야 하기 때문에 매우 시간이 느리고 비효율적입니다. 따라서 Task끼리의 데이터를 공유하는 일은 되도록 지양하지만, Airflow를 사용하다보면 Task끼리 간단한 데이터들을 공유해야할 때가 생길 수도 있습니다. 이럴 때 XCom을 사용하여 Key-Value 형태로 데이터를 간단하게 읽고 쓸 수 있습니다.

> 아래는 XCom 에 데이터를 push하고 pull하는 코드 예시입니다.

```python
def _push_data(**context):
	data = "data"
	context["task_instance"].xcom_push(key = "data_id", value = data)

def _pull_data(**context):
	context["task_instance"].xcom_pull(key = "data_id")

push_data = PythonOperator(
	task_id = "push_data",
	python_callable = _push_data
)

pull_data = PythonOperator(
	task_id = "pull_data",
	python_callable = _pull_data
)
```

XCom은 앞서 설명드렸던 Airflow Context에 데이터를 저장하게 됩니다. 따라서 XCom을 사용하는 메소드의 인자에는 항상 키워드 인자로 context가 포함되어야 합니다. 그리고 XCom에 저장하고 싶은 데이터를 context안에 있는 “task\_instance” 라는 key 안에 입력해주며, 이것은 줄여서 “ti”로 입력하셔도 됩니다.

참고로 XCom은 대용량 데이터에는 적합하지 않으며 만약 큰 데이터를 Task 간 공유가 필요하다면 다른 저장소를 사용하는 것을 추천드립니다.

## 더 알아보기

지금까지 Airflow를 정말 간단하게 살펴보았는데요, 앞으로 Airflow에 대해서 더 공부하고 싶은 분들을 위해 공부할 만한 내용들을 추천드리고자 합니다.

### 1. Airflow template

Airflow 내에서 런타임 단계에서 인자를 넣고 싶을 때 매우 유용하게 사용할 수 있습니다. 또한, jinja template을 사용해서 하드 코딩할 수 있는 영역들을 줄일 수 있게 도와줍니다.

### 2. Sensor

특정 상황에서 DAG을 trigger 할 때 사용되며, Operator의 한 종류입니다.

### 3. Custom

Airflow에서는 정말 간단하게 Custom hook, operator, hook을 구현할 수 있습니다.

### 4. Executor

Airflow가 작동하는 원리를 조금 더 이해하고, 운영환경에서 대해서 알고 싶으시면 executor 종류에 대해서 찾아보시길 바랍니다.

Airflow에는 대표적으로 SequentialExecutor, LocalExecutor, CeleryExecutor, KubernetesExecutor가 있습니다.
