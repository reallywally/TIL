# Airflow에 공통 모듈을 어떻게 배포할까

## 배경

airflow dag에서 파이썬 코드를 실행할때 유틸리티 클래스 처럼 공통 모듈이 필요한 경우가 있다. 그리고 지금 프로젝트 환경에서는 flask 서버에서도 공통 모듈이 필요하다. 로컬에서는 특정 폴더에 공통 모듈을 두고 import해서 사용했는데 서버로 올라가면 폴더 위치가 달라진다. 공통 모듈 배포를 고민하면서 해결 과정을 작성해 보자.

## 해결방법

1. setup 파일로 설치  
    공통 모듈 루트 경로에 ```setup.py```을 만들고 배포될 환경에 설치하는 방식이다.

    ```python
    from setuptools import setup, find_packages

    setup(
        name="my-custom-module",
        version="0.5",
        packages=find_packages()
    )
    ```

    ```shell
    python -m pip install .
    ```

2. 특정 위치나 plugin에 복사  
    사실 이 방법으로 작동이 올바르게 실행되는지 테스트해보진 않았다. 하지만 리서칭 해보면 항상 나오는 방법이고 특별히 안될만한 이유도 없을것 같아 메모해두고 나중에 여유될때 테스해볼 예정이다.

    ```markdown
    dags/
    ├── my_dag.py
    ├── my_module.py
    my_custom_module/
    └── my_module1.py
    plugins/
    └── my_plugin/
        └── my_module2.py
    ```

    ```python
    # my_dag.py
    from airflow import DAG
    from airflow.operators.python import PythonOperator
    from datetime import datetime
    from my_custom_module.my_module1 import process_data  # 사용자 정의 모듈 import

    with DAG(
        dag_id="example_custom_module",
        schedule_interval=None,
        start_date=datetime(2025, 1, 1),
        catchup=False,
    ) as dag:
        process_task = PythonOperator(
            task_id="process_data_task",
            python_callable=process_data,
        )
    ...
    ```

## 참고자료

* no module 문제 해결하기: <https://hongcana.tistory.com/156>
