# Feature Store란 무엇인가? ML 엔지니어가 알아야 할 핵심 개념

## 🤔 이런 상황, 익숙하지 않나요?

데이터 사이언티스트 A: "고객의 최근 30일 구매 금액 feature가 필요해요"  
데이터 엔지니어 B: "아, 그거 지난주에 팀 C에서도 만들었는데..."  
ML 엔지니어 C: "학습할 때는 잘 되던 모델이 실서비스에서는 왜 이상하게 동작하지?"  

이런 상황들이 반복되면서 ML 팀은 점점 더 많은 시간을 **동일한 feature를 반복 개발**하고, **학습-서빙 간 불일치**를 디버깅하는 데 소모하게 됩니다.

**Feature Store**는 바로 이런 문제들을 해결하기 위해 등장한 ML 인프라입니다.

## 📊 Feature Store란 무엇인가?

**Feature Store**는 머신러닝 feature를 **중앙화된 저장소**에서 관리하고 제공하는 시스템입니다.

간단히 말하면:

- **Feature**: 모델이 예측에 사용하는 입력 데이터 (고객 나이, 구매 이력 등)
- **Store**: 이런 feature들을 효율적으로 저장하고 관리하는 중앙화된 시스템

마치 **코드의 라이브러리**처럼, 한 번 만들어진 feature를 여러 모델과 팀에서 재사용할 수 있게 해주는 **"Feature의 라이브러리"**라고 생각하면 됩니다.

## 🎯 왜 Feature Store가 필요할까?

### 1. Feature 개발의 중복 문제

```python
# 팀 A의 코드
def calculate_user_purchase_amount_30d(user_id):
    # 30일간 구매 금액 계산 로직
    return amount

# 팀 B의 코드 (같은 기능, 다른 구현)
def get_recent_purchase_sum(customer_id):
    # 거의 동일한 로직을 다시 구현
    return total
```

**문제점**: 같은 feature를 여러 팀이 각각 개발하여 **개발 비용 증가**

### 2. Training-Serving Skew

```python
# 학습 시 (오프라인)
training_features = pd.read_sql("SELECT AVG(amount) FROM purchases WHERE date >= '2023-01-01'")

# 서빙 시 (온라인) - 다른 로직!
serving_features = redis.get(f"user_avg_purchase_{user_id}")
```

**문제점**: 학습과 서빙에서 **다른 feature 생성 로직**으로 인한 성능 저하

### 3. 데이터 일관성 문제

각 팀이 서로 다른 데이터 소스를 사용하거나 다른 시점의 데이터를 사용하여 **일관성 없는 feature** 생성

## 🏗️ Feature Store의 핵심 구성 요소

### 1. 온라인 스토어 (Online Store)

**목적**: 실시간 추론을 위한 저장소

```python
# 실시간 추론 시 사용
features = online_store.get_features(
    feature_service="user_features",
    entity_id="user_12345"
)
# 응답 시간: 1-10ms
```

**특징**:

- 🚀 **초저지연**: 1-10ms 내 응답
- 🔑 **Key-Value 구조**: 빠른 조회를 위한 NoSQL
- 📊 **최신 데이터**: 현재 시점의 feature 값
- 🎯 **기술 스택**: Redis, DynamoDB, Cassandra

### 2. 오프라인 스토어 (Offline Store)

**목적**: 모델 학습을 위한 대용량 데이터 처리

```python
# 모델 학습 시 사용
training_data = offline_store.get_historical_features(
    feature_service="user_features",
    entity_df=user_list,
    start_date="2023-01-01",
    end_date="2023-12-31"
)
# 처리량: 테라바이트급 데이터
```

**특징**:

- 📈 **대용량 처리**: 테라바이트급 데이터 처리
- ⏰ **시점 정확성**: 과거 특정 시점의 정확한 feature 값
- 🔄 **배치 처리**: 효율적인 일괄 처리
- 🎯 **기술 스택**: Parquet, BigQuery, Snowflake

### 3. Feature Registry

**목적**: Feature의 메타데이터 관리

```yaml
# Feature 정의 예시
user_purchase_features:
  entities: [user_id]
  features:
    - avg_purchase_amount_30d: 최근 30일 평균 구매 금액
    - purchase_count_7d: 최근 7일 구매 횟수
  data_source: user_transactions
  refresh_interval: 1hour
```

## 🚀 실제 사용 예시: 상품 추천 시스템

### 시나리오

온라인 쇼핑몰에서 사용자에게 개인화된 상품을 추천하는 시스템

### 1. Feature 정의

```python
# 사용자 feature
user_features = FeatureView(
    name="user_features",
    entities=[user],
    features=[
        Feature(name="age", dtype=Int64),
        Feature(name="avg_purchase_amount_30d", dtype=Float64),
        Feature(name="preferred_category", dtype=String),
    ],
    source=user_source,
    ttl=timedelta(hours=24)
)

# 상품 feature
item_features = FeatureView(
    name="item_features", 
    entities=[item],
    features=[
        Feature(name="category", dtype=String),
        Feature(name="popularity_score", dtype=Float64),
        Feature(name="avg_rating", dtype=Float64),
    ],
    source=item_source,
    ttl=timedelta(hours=6)
)
```

### 2. 모델 학습 시

```python
# 과거 데이터로 학습
training_df = store.get_historical_features(
    entity_df=user_item_pairs,  # 사용자-상품 페어 리스트
    features=[
        "user_features:age",
        "user_features:avg_purchase_amount_30d", 
        "item_features:category",
        "item_features:popularity_score"
    ],
    start_date="2023-01-01",
    end_date="2023-11-30"
)

# 모델 학습
model.fit(training_df)
```

### 3. 실시간 추론 시

```python
# 실시간 추천 API
@app.route('/recommend/<user_id>')
def recommend(user_id):
    # Feature Store에서 실시간 feature 조회
    features = store.get_online_features(
        feature_service="recommendation_features",
        entity_rows=[{"user_id": user_id}]
    )
    
    # 모델 추론
    recommendations = model.predict(features)
    return jsonify(recommendations)
```

## 📈 Feature Store의 장점

### 1. 개발 효율성 향상

- ✅ **재사용성**: 한 번 개발된 feature를 여러 모델에서 활용
- ✅ **표준화**: 일관된 feature 정의로 개발 시간 단축
- ✅ **협업**: 팀 간 feature 공유 및 지식 전파

### 2. 모델 성능 향상

- ✅ **일관성**: 학습과 서빙에서 동일한 feature 로직 사용
- ✅ **최신성**: 실시간 feature 업데이트로 모델 성능 유지
- ✅ **품질**: 중앙화된 검증 및 모니터링

### 3. 운영 효율성

- ✅ **자동화**: 복잡한 feature 파이프라인 자동 관리
- ✅ **모니터링**: 통합된 feature 상태 모니터링
- ✅ **백필**: 새로운 feature 추가 시 과거 데이터 자동 생성

## ⚠️ 도입 시 고려사항

### 1. 언제 도입해야 할까?

**도입 권장 상황**:

- 여러 ML 모델을 운영하는 조직
- 팀 간 동일한 feature 중복 개발이 빈번한 경우
- Training-serving skew 문제가 자주 발생하는 경우

**도입 비권장 상황**:

- 단일 모델만 운영하는
 소규모 팀
- Feature가 매우 단순하고 적은 경우
- 실시간 추론 요구사항이 없는 경우

### 2. 기술적 고려사항

```python
# 네트워크 지연 고려
# 기존: 직접 DB 조회 (지연시간 적음)
features = db.query("SELECT * FROM user_features WHERE id = %s", user_id)

# Feature Store: 추가 네트워크 호출 (지연시간 증가)
features = feature_store.get_features(entity_id=user_id)
```

**주의점**:

- 📊 **성능 오버헤드**: 추가 네트워크 호출로 인한 지연
- 🔧 **운영 복잡성**: 추가 인프라 관리 필요
- 💰 **비용**: 고성능 온라인 스토어 운영 비용

## 🛠️ 주요 Feature Store 솔루션

### 오픈소스

- **Feast**: 가장 인기 있는 오픈소스 Feature Store
- **Feathr**: LinkedIn에서 개발한 엔터프라이즈급 솔루션
- **Hopsworks**: 완전한 MLOps 플랫폼 포함

### 관리형 서비스

- **AWS SageMaker Feature Store**: AWS 생태계 통합
- **Google Vertex AI Feature Store**: GCP 환경 최적화
- **Azure Machine Learning Feature Store**: Microsoft 클라우드 서비스

## 🎯 마무리: Feature Store, 도입해야 할까?

Feature Store는 **ML 조직의 성숙도가 높아질수록 필수적**인 인프라입니다.

**도입을 고려해보세요**:

- 👥 여러 팀이 ML 모델을 개발하고 있다
- 🔄 동일한 feature를 반복 개발하고 있다
- 🐛 Training-serving skew 문제를 자주 경험한다
- 📈 모델 수와 복잡성이 지속적으로 증가한다

**신중하게 접근하세요**:

- 🎯 명확한 ROI 계산 후 도입
- 🏗️ 점진적 도입 (핵심 feature부터 시작)
- 👨‍💻 팀의 기술 역량과 운영 능력 고려

Feature Store는 **"ML 팀의 생산성을 높이는 인프라"**입니다.
단순히 기술적 솔루션이 아닌, **조직의 ML 워크플로우를 개선하는 도구**로 접근해야 합니다.
