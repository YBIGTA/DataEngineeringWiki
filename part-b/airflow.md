# Airflow 개요

## ![](<../.gitbook/assets/airflow\_overview\_1 (1).png>)

## Airflow 소개

Airflow는 데이터 파이프라인을 처리하기 위한 배치 테스크(batch-oriented framework)에 중점을 둔 프레임워크입니다. 여기서 배치 테스크란 데이터를 실시간으로 처리하는 것이 아니라, 일괄적으로 모아서 처리하는 작업을 의미합니다. 보통 데이터 파이프라인 하면 아무래도 ‘파이프라인’이라는 단어 때문에 실시간 데이터를 떠오르시는 분들이 많을 것 같은데요, 생각보다 배치 작업이 필요한 테스크들이 많이 있습니다. 예를 들어 하루에 한 번 업데이트 되는 대시보드를 만들고 싶을 때, 주기적으로 가공된 데이터를 새로운 모델에 넣어 학습시키고 싶을 때 에어플로우를 사용할 수 있습니다.

## Airflow의 등장

2014년에 Airbnb의 엔지니어들은 회사 내에서 여러 시스템에서 작업을 조율하고 복잡한 데이터 워크플로를 관리하는 데 어려움에 직면했습니다. 서로 다른 시스템의 작업을 조율하고 분석이나 배포시간에 맞추어 적절하게 작동하는 파이프라인을 만들기란 여간 어려운 일이 아니었죠. 그래서 이들은 Workflow를 작성 및 예약하고 웹 인터페이스를 기본으로 제공하여 워크플로 운영을 모니터링 할 수 있는 오픈 소스 솔루션인 Airflow를 개발했습니다.

## 왜 Airflow인가 ?

1. Extensible

앞서 ‘Airflow의 등장’ 에서도 눈치채셨을 수 있지만 Airflow를 사용하는 가장 큰 이유 중 하는 다른 시스템 간 연결하는 작업을 매우 쉽게 할 수 있다는 것입니다. 데이터 엔지니어링이나 DB에 대해 공부를 조금이라도 해보셨으면 들었을 법한 AWS S3, GCP BigQuery, MongoDB, Postgresql 등등 데이터와 관련된 시스템에 Airflow를 사용한다면 매우 쉽게 연결하여 원하는 작업을 수행할 수 있습니다.

1. Dynamic

저희는 상황에 따라서 변하는 성질을 Dynamic(동적)이라고 하죠, Airflow는 저희에게 익숙한 파이썬을 활용하여 상황에 따라 다르게 돌아가는 파이프라인을 생성할 수도 있습니다. 즉, 원한다면 파이썬 언어에서 구현하는 방법을 사용하여 커스텀 파이프라인을 생성할 수 있습니다.

1. Monitoring

Airflow는 UI를 제공하여 데이터 파이프라인에서 사용되는 여러 기능들을 쉽게 모니터링 가능합니다. 파이프라인에서 문제가 생길 시 어디에서 문제가 생긴지 빠르게 파악하고 로그를 통해 구체적인 내용들을 확인하여 대처할 수 있는 것이죠.

<figure><img src="../.gitbook/assets/airflow_overview_2 (1).png" alt=""><figcaption><p><a href="https://airflow.apache.org/docs/apache-airflow/stable/index.html">https://airflow.apache.org/docs/apache-airflow/stable/index.html</a></p></figcaption></figure>

1. Scheduling

수 많은 스케줄링 기법은 파이프라인을 정기적으로 실행하고 증분 처리를 통해, 전체 파이프라인을 재실행할 필요 없는 효율적인 파이프라인 생성이 가능합니다.

## DAG

Airflow는 Workflow 구축하고 실행할 수 있는 플랫폼이고 Workflow 내부의 각 작업들을 DAG(Directed Acyclic Graph)를 통해 구조화 합니다. DAG이란 아래와 같이 순환하는 싸이클이 존재하지 않고, 일방향성만을 갖고 있는 비순환 구조로 되어 있는 그래프를 의미합니다.

<figure><img src="../.gitbook/assets/airflow_overview_3 (1).png" alt=""><figcaption></figcaption></figure>

위에서 언급했듯이 Airflow는 Worflow를 오케스트레이션하는 프로그램이므로 EDA(Event Driven Architecture)로서 각 작업들은 DAG의 구조를 보이게 됩니다. DAG는 작업 간의 종속성과 이를 실행하는 순서를 지정합니다. 작업 자체는 데이터 가져오기, 분석 실행, 다른 시스템 트리거 등 무엇을 해야 하는지 설명합니다.아래는 Ariflow 공식 문서에 나와 있는 DAG 구조의 예시입니다.

<figure><img src="../.gitbook/assets/airflow_overview_4 (1).png" alt=""><figcaption><p><a href="https://airflow.apache.org/docs/apache-airflow/stable/_images/edge_label_example.png">https://airflow.apache.org/docs/apache-airflow/stable/_images/edge_label_example.png</a></p></figcaption></figure>

위 예시에서는 첫 번째 작업인 ingest부터 마지막 작업인 report까지 순차적으로 작업이 실행됩니다. 중간에 check\_integrity라는 이름의 작업은 Error를 발생을 기준으로 분기하여 Task를 진행합니다.

## Summary

정리하자면 Airflow는 데이터 파이프라인에서 사용되고 있는 여러 시스템과 프레임워크, 기능들을 자동으로 설정, 관리, 조정을 해주는 데이터 오케스트레이션 프레임워크라고 할 수 있습니다. Airflow가 있기 전에는 모든 작업들을 하나의 애플리케이션으로 구현하여 작업했고 각 작업을 수정해야 하면 전체 애플리케이션을 수정 해야하는 번거로움이 있었습니다. 그러나 Airflow를 사용하면 각 작업들을 독립적으로 수행하고 이에 대한 실행 순서를 조정하는 것은 Airflow를 통해 DAG 구조하에서 간단하게 구현이 가능하게 된 것입니다.
