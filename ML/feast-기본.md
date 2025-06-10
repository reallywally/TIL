# feast 기본

## feast란

Feature Store의 약자로 feast라 하며 Feature Store 개념에 필요한 기능을 제공하는 오픈소스 소프트웨어이다.

## feast의 주요 역할

- Feature Store: 머신러닝 프로젝트에서 여러 데이터 사이언티스트와 엔지니어가 만든 다양한 피처(컬럼, 변수)를 한 곳에 모아 관리한다. 이를 통해 피처의 재사용성과 일관성을 높이고, 팀 간 협업을 원활하게 할수 있다.
- 실시간 및 배치 데이터 지원: Feast는 오프라인(배치) 데이터뿐만 아니라 온라인(실시간) 데이터도 관리할 수 있어, 실시간 예측 서비스에도 활용된다.
- ML 인프라와 분리: 모델 개발과 인프라를 분리하여, 데이터 접근 계층을 단순화하고 ML 파이프라인의 유지보수를 쉽게 만든다.

## feast를 사용하는 이유

- 여러 팀이 동일한 피처를 반복해서 만들 필요 없이, 한 번 정의된 피처를 저장소에서 쉽게 찾아서 재사용할 수 있다,
- 모델 학습과 서빙(실시간 예측)에 필요한 피처를 일관성 있게 제공할 수 있다.
- 데이터 소스(데이터 레이크, 데이터 웨어하우스 등)와 ML 모델 사이의 연결을 표준화하여, 데이터 엔지니어링과 ML 개발을 분리할 수 있다.

## feast 컨셉

정확하진 않지만 개념적인 구조와 의미는 다음과 같다.

```text
  ├── Feature View(테이블) ──── Feature Service(조인)
       ├── Feature(컬럼)
       ├── Entity(PK)
       ├── Data Source(DB)
```

## 궁금했던 내용 정리

1. 아래와 같이 store를 2개 만들면 각각은 독립적인 feast(Feasture Store)인가?

    ```python
    from feast import FeatureStore

    store1 = FeatureStore()
    store2 = FeatureStore()
    ```

    -> 아니다. FeatureStore는 설정 파일(feature_store.yaml)을 기준으로 인스턴스 되기 때문에 store1, store2는 같은 스토어이다. 만약 독립적인 feast를 만들려면 아래와 같이 설정 파일을 분리하면 된다.

    ```python
    from feast import FeatureStore, Config

    store1 = FeatureStore(repo_path="feature_store_spark.yaml")
    store2 = FeatureStore(repo_path="feature_store_bigquery.yaml")
    ```

2. 오프라인 스토어로 연결된 DB를 직접 수정하면 apply와 동일한 결과를 볼 수 있을끼?

    단순히 feature명이나 데이터에 오타가 있어서 수정하다가 궁금했다.  
    -> 아니다. 기능적으로는 변경할수 있으나 feast가 온라인 스토어를 위해 materialized를 해야하기 때문에 전체 로직을 실행시키면 동일한 결과를 보기 어려울 수 있다. 그리고 apply는 메타데이터를 Registry db에 반영하는데 이 Registry db와도 싱크가 안맞을 수 있다.  
    비즈니스 로직을 개발 할때도 1개의 API 요청으로 여러 테이블이 연결되기 때문에 DB로 직접 값을 변경하는건 위험하다. feast 역시 마찬가지로 DB로 직접 접근해서 값을 수정하는 작업은 하지 않는것이 좋다.

3. 온라인 스토어와 메타 데이터는 메모리에 저장이되나?

    아니다. materialize를 오프라인 스토어에서 온라인으로 적재하는것이고, 온라인 스토어 예시로 redis가 있어서 메모리에 올라간다고 잘못 이해하고 있었다. 하지만 1번 내용과같이 설정 정보는 yaml에있고, yaml에 메타정보를 저장하는 registry랑 온라인 스토어 저장 정보가 따로 있다.

## 참고자료

- <https://heon28.tistory.com/16>
