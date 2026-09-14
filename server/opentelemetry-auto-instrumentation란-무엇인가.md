# TIL: OpenTelemetry Auto Instrumentation (Python / FastAPI)

## 1. OTEL(OpenTelemetry)이 뭐야?

서버가 "지금 무슨 일을 하고 있는지"를 기록하고 밖으로 내보내는 **공통 규칙**이다.

비유하면 이렇다. 택배 하나가 물류센터 여러 곳을 거쳐 배송된다. 어디서 얼마나 머물렀는지 알고 싶으면 각 센터마다 스캔 기록이 남아야 한다. OTel은 서버 세계의 "스캔 기록을 남기는 표준 방식"이다. 어느 회사 스캐너를 쓰든 기록 형식이 같아서, 나중에 조회 시스템을 바꿔도 기록은 그대로 쓸 수 있다.

기록하는 것 세 종류:

| 종류 | 뭘 기록하나 | 예시 |
| --- | --- | --- |
| **Trace** | 요청 하나가 거쳐간 경로와 각 단계의 소요 시간 | "이 API 요청은 DB 조회 200ms + LLM 호출 1.5초 걸렸다" |
| **Metric** | 시간에 따른 숫자 | "1분 동안 요청 300건, 그중 에러 5건" |
| **Log** | 특정 순간의 이벤트 메시지 | "유저 123 로그인 실패" |

Trace의 최소 단위가 **span**이다. "DB 조회"가 span 하나, "LLM 호출"이 span 하나, 이것들이 모여서 요청 하나의 trace가 된다.

### 구성 요소 (등장인물)

- **API / SDK**: 우리 코드 안에서 기록을 만드는 라이브러리.
- **Exporter**: 만든 기록을 밖으로 보내는 부품.
- **Collector**: 여러 서버에서 온 기록을 한데 모아 정리한 뒤, 저장소로 넘겨주는 중간 서버. 없어도 되지만 있으면 편하다.
- **백엔드**: 기록을 저장하고 화면에 보여주는 곳. Jaeger, Grafana Tempo, Azure Monitor 등.

흐름: `내 FastAPI 서버 → (Exporter) → Collector → Azure Monitor / Jaeger`

### 왜 쓰나

- 코드는 OTel 방식 하나로 기록하고, 저장소는 나중에 마음대로 바꿀 수 있다. (벤더 종속 탈출)
- 서비스가 여러 개여도 요청 하나를 처음부터 끝까지 이어서 볼 수 있다.

## 2. `opentelemetry-api`와 `opentelemetry-sdk`는 왜 둘로 나뉘어 있어?

- **api**: "기록을 남긴다"는 **동작의 이름만** 정해둔 껍데기. 혼자 있으면 실제로는 아무것도 안 한다.
- **sdk**: 그 이름에 **실제 동작**을 붙여주는 본체.

플러그와 콘센트 관계다. api는 플러그 모양(규격), sdk는 전기가 실제로 흐르는 콘센트다. 플러그만 있으면 꽂을 곳이 없어서 아무 일도 안 일어난다.

왜 이렇게 나눴나: FastAPI, httpx 같은 라이브러리들이 "우린 플러그만 달아둘게, 콘센트는 앱 만드는 사람이 알아서"라고 할 수 있어야 하기 때문. 콘센트(sdk)가 없는 환경에서는 그냥 조용히 넘어가서 라이브러리 입장에서 부담이 없다.

> `pip freeze`에 `opentelemetry-api`만 있다면? 어떤 패키지가 플러그만 달아둔 것이다. sdk를 설치하고 설정하지 않았다면 기록은 **하나도 수집되고 있지 않다.**

## 3. Auto Instrumentation이 뭐야?

**Instrumentation** = 기록 남기는 코드를 심는 것.

- **Manual**: 내가 직접 코드에 `tracer.start_span("DB 조회")` 같은 걸 한 줄씩 쓴다.
- **Auto**: 내가 안 써도 라이브러리가 대신 심어준다.

FastAPI로 예를 들면, auto instrumentation은 "모든 API 요청이 들어올 때 자동으로 span을 만들고, 어떤 URL이었는지·몇 ms 걸렸는지·상태 코드가 뭐였는지 알아서 붙여주는" 것이다. 내 엔드포인트 함수 코드는 한 글자도 안 바뀐다.

내부적으로는 FastAPI 앞에 미들웨어(요청이 지나가는 검문소)를 하나 끼워서 동작한다.

## 4. 어떻게 쓰나 — 두 가지 방법

### 방법 A: 코드에 한 줄 추가

```bash
pip install opentelemetry-instrumentation-fastapi opentelemetry-sdk opentelemetry-exporter-otlp
```

```python
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor

app = FastAPI()
FastAPIInstrumentor.instrument_app(app)   # 이 한 줄
```

이 한 줄이면 모든 엔드포인트가 기록된다. 단, 어디로 보낼지(exporter) 설정은 따로 해줘야 한다.

### 방법 B: 코드는 안 건드리고 실행 명령만 바꾸기 (zero-code)

`main.py`는 그대로 두고, Dockerfile과 환경변수만 바꾼다.

```dockerfile
# 1) 패키지 설치
RUN pip install opentelemetry-distro opentelemetry-exporter-otlp \
 && opentelemetry-bootstrap -a install

# 2) 실행 명령 앞에 opentelemetry-instrument 붙이기
CMD ["opentelemetry-instrument", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

- `opentelemetry-bootstrap -a install`: "내 프로젝트에 fastapi, httpx, sqlalchemy가 있네? 각각에 맞는 자동 계측 패키지를 깔아줄게"를 대신 해준다.
- `opentelemetry-instrument`: 파이썬이 켜지는 순간 끼어들어서 "설치된 자동 계측 다 켜라"를 실행한다.

설정은 전부 환경변수로 넘긴다. (K8s Deployment의 env, ConfigMap, GitLab CI 변수 등)

```yaml
OTEL_SERVICE_NAME: agent-server                          # 이 서버 이름
OTEL_EXPORTER_OTLP_ENDPOINT: http://otel-collector:4317  # 기록 보낼 곳
OTEL_TRACES_EXPORTER: otlp
OTEL_RESOURCE_ATTRIBUTES: deployment.environment=prod
```

### 방법 C: (Kubernetes) Dockerfile도 안 건드리기

OpenTelemetry Operator를 클러스터에 깔면, Pod에 표시 하나만 붙여도 Operator가 알아서 방법 B를 대신 해준다.

```yaml
metadata:
  annotations:
    instrumentation.opentelemetry.io/inject-python: "true"
```

앱 저장소는 전혀 수정 없이 인프라 쪽 설정만으로 끝난다.

## 5. 헷갈리기 쉬운 것

**"FastAPI만 깔면 되는 거 아냐?"**
아니다. FastAPI 안에는 OTel 코드가 하나도 없다. 어떤 방법을 쓰든 pip 설치는 필요하다.

**"zero-code면 저장소 안 건드려도 되는 거지?"**
"앱 코드"는 안 건드리지만 "저장소"는 건드린다. requirements나 Dockerfile, 실행 명령은 바꿔야 한다. (방법 C는 예외)

**"FastAPI 계측했으면 LLM 호출도 다 보이겠지?"**
안 보인다. FastAPI 계측은 **들어오는 요청**만 잡는다. 서버가 밖으로 보내는 요청(OpenAI API, 다른 마이크로서비스)까지 이어서 보려면 그 요청을 보내는 라이브러리용 계측도 깔아야 한다.

```bash
pip install opentelemetry-instrumentation-httpx   # httpx 쓴다면
```

이걸 빼먹으면 trace가 "요청 들어옴 → ??? → 응답 나감"으로 중간이 비어 보인다.

## 6. 한계와 주의점

- **자동 계측은 라이브러리 경계만 잡는다.** HTTP 요청, DB 쿼리, 외부 API 호출은 보이지만, 내 함수 안에서 "왜 여기가 느린지"는 안 보인다. 그건 manual span을 직접 넣어야 한다.
- **uvicorn `--workers N`** 으로 여러 프로세스를 띄우면 워커에 계측이 안 걸리는 경우가 있다. K8s에서는 워커 1개 + replica로 늘리는 게 안전하다.
- **Azure Monitor로 바로 보내지 말고 Collector를 거치자.** 나중에 저장소를 바꿀 때 Collector 설정만 고치면 된다.

## 한 줄 요약

자동 계측으로 바닥(요청/DB/외부호출)을 깔고, 진짜 궁금한 구간에만 수동 span을 얹는다.

## 기타

fable 5.1이 97% 작성해주었다.
