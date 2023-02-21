# Airflow 구조

## Airflow의 구조

공식 문서에 따르면 Airflow는 아래와 같은 구조를 갖고 있습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/80750cdf-1ee5-45da-b49c-095b2a86eae0/Untitled.png)

```
         [<https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html>](<https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html>)
```

* Scheduler : 예약된 Worflow를 Trigger하고 실행할 작업을 실행기에 제출하는 작업을 모두 처리합니다.
* Executor : Executor에서는 실행 중인 작업을 처리합니다.
* Webserver : DAG 및 작업의 동작을 검사, 트리거 및 디버그 하기 위한 User Interface를 제공합니다.
* Meta Database : Scheduler 및 Webserver에서 상태를 저장하는 기능을 수행합니다.
* DAG Directory : Scheduler와 Executor가 읽는 DAG 파일을 모아두는 곳입니다.

이 구성 요소들이 작동되는 flow는 아래와 같습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3f85b7f8-6a45-4d3d-99b7-19a7301bd618/Untitled.png)

1. 사용자가 DAG Workflow를 작성하면, 스케줄러는 DAG 파일을 분석하고 각 DAG 태스크, 의존성 및 예약 주기를 확인합니다.
2. 스케줄러는 마지막 DAG까지 내용을 확인한 후 DAG의 예약 주기가 경과 했는지 확인합니다. 예약 주기가 현재 시간 이전이라면 실행되도록 예약합니다.
3. 예약된 각 태스크에 대해 스케줄러는 해당 태스크의 의존성(=업스트림 태스크)을 확인합니다. 의존성 태스크가 완료되지 않았다면 실행 대기열에 추가합니다.
4. 스케줄러는 1단계로 다시 돌아간 후 새로운 루프를 잠시 동안 대기합니다.

## Operator

Airflow를 사용한 파이프라인에서는 하나 이상의 단계로 구성된 대규모 작업을 개별 태스크로 분할하고 DAG로 형성합니다. 이때 각 태스크를 정의할 때 필요한 것이 Operator입니다. (사실상 Operator와 태스크는 사용자 관점에서는 같은 의미입니다. 그러나 조금 분리해서 설명드리자면 코드로 작성된 Operator가 실행되면 하나의 태스크가 생성 된다고 이해하시면 될 것 같습니다.) 결론적으로 Airflow DAG 내부에서 의존성이 부여된 Operator가 실행되는 것이 코드의 관점에서 Airflow가 동작하는 방식입니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8cf4c1a4-0f20-451a-946f-59ceb1728da0/Untitled.png)

## 의존성 부여

Airflow에서 태스크 사이에 실행 순서, 즉 의존성을 부여하는 것은 매우 쉽습니다. 바로 오른쪽 시프트 연산자(>>) 를 사용하여 정의하면 됩니다. 아래의 예시들을 보시면 쉽게 이해하실 수 있습니다.

> 예시 1

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/848a46ae-bdb6-4f5e-b6bd-8ff33a603917/Untitled.png)

```python
Task1 >> Task2 >> Task3
```

> 예시 2

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1c981c52-f6d3-4ba5-ac79-3aa392b79f08/Untitled.png)

```python
Task1 >> [Task2, Task3] >> Task4
```

## Airflow 구현

앞에서 DAG, Operator, 의존성 부여에 대해 설명해드렸습니다. 이를 기반으로 이번에는 매우 간단한 코드 예시를 통해 코드로서 이러한 개념들이 어떻게 구현되는지 살표보고 앞에서 이해 못했던 내용들을 채워보도록 합시다. 먼저 저희가 구현하고자 하는 Workflow 입니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/65ee8aed-f99f-41cb-a6f7-bc5185e31cb7/Untitled.png)

이러한 Worflow를 구현하기 위한 코드입니다. 참고로, 구체적인 파이썬 구현과 Airflow를 위해 입력해야할 설정 값들은 모두 생략했습니다.

```python
import airflow
from airflow import DAG
from airflow.operators.python import PythonOperator

def _get_weather_data():
	# 코드 생략
	pass

def _data_preprocessing():
	# 코드 생략
	pass

def _load_to_database():
	# 코드 생략
	pass

test_dag = DAG(
	dag_id = "test_dag_for_YBIGTA_DE"
	start_date = ...
	schedule_interval = ...
)

get_weather_data = PythonOperators(
	task_id = "get_weather_data",
	python_callable = _get_weather_data,
	dag = test_dag
)

data_preprocessing = PythonOperators(
	task_id = "data_preprocessing",
	python_callable = _data_preprocessing,
	dag = test_dag
)

load_to_database = PythonOperators(
	task_id = "load_to_database",
	python_callable = _load_to_database,
	dag = test_dag
)

get_weather_data >> data_preprocessing >> load_to_database
```

이제 구체적으로 코드를 살펴보도록 하겠습니다.

> 필요한 모듈 import

```python
import airflow
from airflow import DAG
from airflow.operators.python import PythonOperator
```

먼저 필요한 모듈들을 불러옵니다.

> 파이썬 함수 정의

```python
def _get_weather_data():
	# 코드 생략
	pass

def _data_preprocessing():
	# 코드 생략
	pass

def _load_to_database():
	# 코드 생략
	pass
```

각 Task에서 기능을 수행할 파이썬 함수를 정의합니다. 예시에서는 같은 파이썬 파일에 작성되어 있지만 보통은 매우 간단한 기능이 아니라면 패키지화 하여 불러옵니다.

> DAG 정의

```python
test_dag = DAG(
	dag_id = "test_dag_for_YBIGTA_DE"
	start_date = ...
	schedule_interval = ...
)
```

test\_dag 라는 이름으로 DAG를 구체화한 인스턴스를 만들었습니다. dag\_id라는 인자에 해당 dag의 id 명을 부여했습니다. 또한 start\_date 에 DAG를 처음 실행을 시작할 날짜를 입력했고, schedule\_interval 인자에는 DAG의 실행 간격에 대한 설정 값을 부여했습니다. 언급된 인자 외에도 DAG에는 여러가지 설정 값들을 부여할 수 있습니다.

> Operator 생성

```python
get_weather_data = PythonOperator(
	task_id = "get_weather_data",
	python_callable = _get_weather_data,
	dag = test_dag
)

data_preprocessing = PythonOperator(
	task_id = "data_preprocessing",
	python_callable = _data_preprocessing,
	dag = test_dag
)

load_to_database = PythonOperator(
	task_id = "load_to_database",
	python_callable = _load_to_database,
	dag = test_dag
)
```

Airflow에서 제공하는 Operator의 종류는 굉장히 다양하지만 해당 예시에서는 파이썬 스크립트를 실행할 수 있는 PythonOperator를 사용했습니다. 앞서 DAG 인스턴스를 생성했던 것과 비슷하게 id 명을 task\_id 인자에 입력합니다. 또한, python\_callable 인수에 해당 Operator에서 수행할 파이썬 함수를 불러옵니다. 마지막으로 dag 인자에 Operator가 어떤 DAG 안에서 동작할 것인지 정의해줍니다. 이 외에도 다양한 인수들을 사용하여 Operator의 사용성을 정의할 수 있습니다.

> 의존성 부여

```python
get_weather_data >> data_preprocessing >> load_to_database
```

저희는 세 가지 태스크가 순차적으로 진행되는 DAG을 작성하고 싶기 때문에 위와 같이 직렬로 진행되는 태스크를 간단하게 정의할 수 있습니다.

## Airflow UI

Airflow 는 앞에서 설명했듯이 DAG를 확인하고 실행 결과에 대해 모니터링이 가능한 웹 인터페이스를 제공합니다. 로그인 후 기본 페이지에 접근하면 최근 실행 결과에 대한 요약과 다양한 DAG에 대한 내용을 확인할 수 있습니다.

> 현재 사용가능한 DAG와 최근 실행 결과에 대한 내용을 보여주는 Airflow의 웹 인터페이스의 메인 페이지

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/54c24ded-2bfe-4ab6-bab7-071d8a0945cd/Untitled.png)

```
                          [<https://airflow.apache.org/docs/apache-airflow/stable/ui.html>](<https://airflow.apache.org/docs/apache-airflow/stable/ui.html>)
```

Airflow 웹 인터페이스의 메인 페이지에서는 DAG 항목 아래에 여러분이 만든 Workflow 이름을 확인할 수 있습니다. 또한, Schdule 항목 아래에는 cron형태로 해당 Workflow의 스케줄을 확인할 수 있고, Recent Tasks 항목에서는 최근 실행한 워크플로 태스크 상태에 대한 정보가 있습니다.

> 시간에 따른 DAG의 실행 결과를 보여주는 Grid View

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edfe1d4c-6523-46a4-98ca-a8f917b1a7cc/Untitled.png)

```
                           [<https://airflow.apache.org/docs/apache-airflow/stable/ui.html>](<https://airflow.apache.org/docs/apache-airflow/stable/ui.html>)
```

DAG의 막대 차트는 DAG이 실행된 시간을 보여주고 아래의 Grid 형태의 내용들은 하나의 Task 의 상태를 의미합니다. GridView를 통해 파이프라인 안에서 Task 들이 어떻게 작동했는지 한 눈에 쉽게 파악할 수 있습니다.

> 태스크 내용과 태스크 간의 의존성을 보여주는 Airflow 웹 인터페이스의 Graph View

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5e339a37-210a-47f1-8d68-a7118f02dd21/Untitled.png)

```
                         [<https://airflow.apache.org/docs/apache-airflow/stable/ui.html>](<https://airflow.apache.org/docs/apache-airflow/stable/ui.html>)
```

개별 DAG의 태스크와 의존성에 대한 Graph View 화면을 제공합니다. 이 View는 태스크 간의 의존성에 대한 세세한 정보를 제공함으로써 DAG의 구조를 파악할 수 있게 해주고, 개별 DAG에 대한 실행 결과를 확인하는 데 유용합니다.

> DAG의 history 내역을 보여주는 Calendar View

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7e560fd3-38a0-4134-9b09-4d56481f16aa/Untitled.png)

```
                          [<https://airflow.apache.org/docs/apache-airflow/stable/ui.html>](<https://airflow.apache.org/docs/apache-airflow/stable/ui.html>)
```

Calendar View는 몇 달이나 몇 년에 걸친 DAG의 실행 history를 제공합니다. 이를 통해 실행되는 DAG의 전반적인 성공, 실패의 추세를 빠르게 파악할 수 있습니다.

이 외에도 Tree, Gantt 등 사용자가 쉽게 모니터링 할 수 있는 페이지를 제공하고 있고, 심지어 Airflow에서 매우 유용하게 활용할 수 있는 몇몇 기능들을 코드를 통해서가 아니라 UI에서 간편하게 사용할 수 있습니다.
