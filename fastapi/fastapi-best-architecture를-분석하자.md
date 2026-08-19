# fastapi-best-architecture를 분석하자

최근 진행하는 fastapi 서버 개발하는데 패키지 구조, DI, DTO 등 뭔가 불편하고 파이썬스럽지 않게 코딩되고 있다는 느낌을 받았다. 다른 사람은 어떻게 했는지 궁금하여 찾다보니 fastapi-best-architecture(<https://github.com/fastapi-practices/fastapi-best-architecture>)라는 깃헙을 발견했다. 과연 이름 그대로 best architecture인지, 내가 본받을 점을 무엇인지 클로드와 함께 분석했다.

## 진짜 best architecture인가

우선 fastapi-best-architecture가 진짜 best architecture인가 확인해 보았다. 클로드가 여러 방면으로 분석했는데 최종 정리하면 다음과 같다.

| 항목 | 평가 |
| --- | --- |
| 구조 일관성 | 상 — 299파일에서 계층 규칙이 안 흔들린다 |
| 트랜잭션 설계 | 상 — service/crud에서 `commit()` 0회. DI로 경계 처리 |
| stateless service | 적절 — FastAPI 관례. 오판했던 부분 |
| 의존성 방향 | 하 — 양방향, 함수 내부 import 15곳 |
| 자원 라이프사이클 | 하 — import 부작용 + `sys.exit()` |
| 스키마 관리 | 하 — 마이그레이션 0개 + `create_all` |
| 테스트 | 최하 — 7줄 |

구조 일관성과 트랜잭션 설계는 내가 읽어도 잘되있는 느낌이 들었다. 그리고 스키마 관리와 테스트는 완벽한 프로젝트로 제공되는 코드는 아니기 때문에 조금 부족해도 이해가 된다. 그렇다면 stateless service는 왜 오판 했었고, 의존성 방향, 자원 라이프 사이클은 무엇 때문에 저평가 받았는지 자세히 보자

## stateless service

처음 클로드가 판단 했을때는 DI관련하여 이렇게 평가했다.

```text
4. DI가 없고 전역 싱글턴이 21개입니다

user_service: UserService = UserService()   # 메서드 12개 전부 @staticmethod

클래스가 그냥 네임스페이스입니다. redis_client, *_dao 모두 모듈 전역 import라 교체가 불가능합니다. FastAPI의 Depends가 있는데 DB 세션에만 쓰고 나머지는 안
씁니다 — 이게 1번(테스트 부재)의 직접적 원인입니다.
```

요즘 가장 궁금했던 내용이다. 의존성 주입이 너무 불편했고 스프링 처럼 생각하고 사용하려 하니 fastapi 스럽지 못하다는 느낌이 들었다. 그래서 서비스를 DI가 없이 사용하는게 의도된게 아닐가 싶어 다시 물어 보니 이렇게 대답했다.

```text
확인해 보니 제 지적이 과했습니다. 이건 빠뜨린 게 아니라 의도적으로 그은 선으로 보입니다.
Depends 사용처를 전수 조사했습니다. 66곳 전부가 요청 스코프 / 횡단 관심사입니다:

- 인증: HTTPBearer(), jwt_authentication_verify
- 인가: RequestPermission('sys:user:del'), rbac_verify, superuser_verify
- 데이터 권한: DataPermissionFilter(Dept), filter_data_permission
- 레이트 리밋: RateLimiter(Rate(5, Duration.MINUTE))
- 페이지네이션: pagination_ctx(_CustomPage)
- DB 세션: CurrentSession, CurrentSessionTransaction                       

서비스 주입은 0건. 그리고 코드 제너레이터 템플릿(templates/python/api.jinja, service.jinja)이 같은 규칙을 그대로 찍어냅니다 — 즉 이 프로젝트가 스스로
"정답"으로 못박은 패턴입니다. 실수라면 66곳 중 몇 곳은 흔들렸을 텐데, 예외가 하나도 없습니다.

규칙은 명확합니다: Depends는 HTTP 요청 컨텍스트가 필요한 것에만, 협력 객체는 그냥 import.

그 선택이 합리적인 이유

1. 주입할 상태가 없습니다. UserService는 메서드 12개가 전부 @staticmethod이고 인스턴스 필드가 0입니다. 생성 비용도 라이프사이클도 없는 객체를 DI로 wiring하는
건 순수 보일러플레이트입니다. 사실상 "네임스페이스가 붙은 모듈 함수"고, Python에서 이건 정상적인 표현입니다.

2. FastAPI의 Depends로는 service→service 호출을 못 풉니다. 이게 결정적입니다. 실제로 user_service.reset_password가
password_security_service.save_password_history를 호출합니다. Depends는 엔드포인트 시그니처에서 시작하는 트리라서, 서비스 내부에서 다른 서비스를 꺼낼 수단이
없습니다. 하려면 dependency-injector 같은 컨테이너를 추가로 들여야 하고 — 그건 "의존성 하나 줄이려고 프레임워크 하나 더 얹는" 거래입니다.

3. 시그니처 오염 비용이 실재합니다. 서비스를 주입하면 모든 엔드포인트가 svc: Annotated[UserService, Depends(get_user_service)]를 달아야 하고, 서비스 수만큼
팩토리 함수가 생깁니다. 21개 서비스면 팩토리 21개입니다. 얻는 건 stateless 객체의 교체 가능성 하나뿐입니다.

4. 생태계 주류가 이쪽입니다. tiangolo 본인의 full-stack-fastapi-template도 CRUD를 모듈 함수로 직접 import하고, Depends는 SessionDep/CurrentUser에만 씁니다.
DI 컨테이너를 쓰는 쪽(dependency-injector, wireup, svcs)이 소수파고, 대개 "클린 아키텍처"를 명시적으로 표방하는 저장소들입니다. Java/Spring 감각으로 보면
이상하지만, FastAPI 관례로는 이 프로젝트가 표준입니다.

5. Python은 import를 패치할 수 있습니다. 제가 쓴 "테스트 시 교체 불가"는 틀렸습니다. patch('backend.app.admin.service.user_service.user_dao')로 얼마든지
갈아끼웁니다. 전역 싱글턴이 곧 테스트 불가는 아닙니다 — 그건 생성자 주입이 유일한 봉합점인 언어의 이야기입니다.
```

클로드가 또 실수할 수도 있겠지만 지금은 **"규칙은 명확합니다: Depends는 HTTP 요청 컨텍스트가 필요한 것에만, 협력 객체는 그냥 import."** 를 참고하여 의존성 주입을 사용할 예정이다.

## 의존성 방향

### 먼저, 원래 어떻게 돼야 하나

계층형 아키텍처의 규칙은 딱 하나입니다. **화살표는 한 방향으로만.**

```text
API (HTTP 처리)  →  Service (업무 규칙)  →  CRUD (DB 접근)  →  Model (테이블)
                              ↓
        common / core / utils / database  ← "바닥". 아무도 위를 몰라야 함
```

`common`, `core`, `utils`, `database`는 **모든 게 딛고 서는 바닥**입니다.
바닥은 위층에 뭐가 있는지 몰라야 합니다. 콘크리트 기초가 3층 화장실 위치를 알면 안 되는 것처럼요.

### 그런데 실제로는

바닥이 위층을 알고 있습니다.

| 위치 | 문제 |
| --- | --- |
| `backend/common/security/jwt.py:15` | `common`이 `app.admin.model.User`를 import |
| `backend/common/security/permission.py:16` | `common`이 `app.admin.model.DataRule`을 import |
| `backend/common/security/rbac.py:78` | `common`이 `plugin.casbin_rbac`을 import |
| `backend/core/conf.py:11` | **설정**이 `plugin.settings_source`를 import |
| `backend/utils/dynamic_config.py:35` | `utils`가 `plugin.config.service`를 import |
| `backend/middleware/opera_log_middleware.py:13` | 미들웨어가 `app.admin.service`를 직접 호출 |

### "이게 진짜 문제라는" 결정적 증거

`backend/common/security/jwt.py:200`을 보세요.

```python
async def ...:
    from backend.app.admin.crud.crud_user import user_dao   # ← 함수 "안"에서 import
```

`backend/common/security/rbac.py:78`, `backend/utils/dynamic_config.py:34-35`도 똑같습니다.

**왜 파일 맨 위가 아니라 함수 안에서 import 할까요?**
맨 위에 쓰면 **순환 import(circular import)** 로 앱이 아예 안 뜨기 때문입니다.
`common` → `app` → `common` → ... 무한 루프죠. 그래서 "실행될 때까지 미루는" 꼼수를 쓴 겁니다.

> 함수 안 import가 여러 군데 보이면, 그건 스타일 취향이 아니라 **의존성 방향이 꼬였다는 알람**입니다.

### 계층이 새는 곳 두 군데 더

**(1) Service가 HTTP를 알고 있다** — `backend/app/admin/service/auth_service.py:1`

```python
from fastapi import Request, Response
from starlette.background import BackgroundTasks
```

Service는 "로그인이란 무엇인가"라는 **업무 규칙**을 담는 층입니다.
여기에 `Request`가 들어오면:

- Celery 배치 작업이나 CLI 스크립트에서 재사용 불가 (거기엔 `Request`가 없음)
- 테스트하려면 가짜 `Request` 객체를 만들어야 함
- 나중에 GraphQL·gRPC로 갈아탈 때 Service까지 다 뜯어야 함

**(2) CRUD가 업무 예외를 던진다** — `backend/app/admin/crud/crud_user.py:30`

```python
from backend.common.exception import errors
```

CRUD는 "데이터를 가져오거나 없으면 `None`"만 하면 됩니다.
"없으면 404" 같은 **판단**은 Service의 몫입니다.

### 초보자용 요약: 왜 신경 써야 하나

| 증상 | 실제로 겪게 되는 일 |
| --- | --- |
| 순환 import | 파일 순서 조금 바꿨는데 앱이 안 뜸 |
| 재사용 불가 | 로그인 로직을 배치에서 못 씀 |
| 테스트 불가 | `common` 하나 테스트하려니 DB·모델·플러그인이 전부 딸려옴 |
| 변경 파급 | `User` 모델 필드 하나 바꿨는데 보안 모듈이 깨짐 |

실제로 이 프로젝트 `backend/tests/`에는 `conftest.py`조차 없습니다. 단위 테스트를 붙이기 어려운 구조라는 방증입니다.

### 어떻게 고치나

1. **방향 뒤집기 (의존성 역전, DIP)**
   `common`이 `User`를 직접 알 필요 없습니다. `common`은 "이런 모양이면 된다"는 `Protocol`만 정의하고, 실제 `User`는 `app` 쪽에서 넣어줍니다.

   ```python
   # common/security/types.py  ← common은 이것만 안다
   class UserLike(Protocol):
       id: int
       is_superuser: bool
   ```

2. **Service에서 `Request` 제거**
   API 층에서 필요한 값만 꺼내 평범한 인자로 넘깁니다.

   ```python
   # Before
   async def login(*, request: Request, ...)
   # After
   async def login(*, client_ip: str, user_agent: str, ...)
   ```

3. **CRUD는 데이터만** — `None` 반환, 예외는 Service에서.

4. **규칙을 도구로 강제** — [`import-linter`](https://import-linter.readthedocs.io/)를 CI에 넣으면 "`common`은 `app`을 import 할 수 없다"를 자동으로 막아줍니다. 사람 리뷰로는 절대 못 막습니다.

## 자원 라이프사이클

### 라이프사이클이 뭔가요

객체가 **언제 태어나고, 얼마나 살고, 언제 죽는지**입니다.
웹 앱에서는 보통 세 가지 수명이 있습니다.

| 수명 | 예시 |
| --- | --- |
| 앱 전체 (앱 켜질 때 1번) | DB 커넥션 풀, Redis 클라이언트 |
| 요청 1건 | DB 세션, 현재 로그인 사용자 |
| 호출 1번 | 임시 계산 결과 |

### 이 프로젝트의 문제: 전부 "import 시점"에 태어난다

```python
# backend/database/db.py:121-122
async_engine = create_database_async_engine(get_database_url())   # ← 모듈 읽는 순간 실행
async_db_session = create_database_async_session(async_engine)

# backend/database/redis.py:114
redis_client: RedisCli = RedisCli()

# backend/core/conf.py:370
settings = get_settings()      # + .env 없으면 파일까지 복사함 (conf.py:365)

# backend/app/admin/service/user_service.py:321
user_service: UserService = UserService()

# backend/app/admin/crud/crud_user.py:418
user_dao: CRUDUser = CRUDUser(User)
```

마지막 두 패턴(`xxx_service = XxxService()`, `xxx_dao = CRUDXxx(...)`)은 **거의 모든 서비스/CRUD 파일 끝에** 반복됩니다.

`import backend.database.db` 한 줄만 써도 **DB 엔진과 커넥션 풀이 즉시 생성**됩니다.
앱이 시작하기 전에, 테스트를 수집만 해도, 심지어 문서 생성 스크립트를 돌려도요.

### 더 나쁜 부분: 라이브러리가 프로세스를 죽인다

```python
# backend/database/db.py:73  /  backend/database/redis.py:57,60,63
except Exception as e:
    log.error(f'데이터베이스 연결 실패 {e}')
    sys.exit()          # ← 예외를 던지는 게 아니라 프로세스를 종료
```

`sys.exit()`는 **애플리케이션 진입점(main)만** 할 수 있는 일입니다.
바닥 모듈이 이걸 하면 호출한 쪽은 재시도도, 대체 동작도, 에러 메시지 가공도 못 합니다. 그냥 죽습니다.

### "생성"과 "초기화"가 따로 논다

`backend/core/registrar.py:44`에는 제대로 된 `lifespan`이 있습니다.

```python
@lifespan_manager.register
async def register_init(app: FastAPI):
    await create_tables()
    await redis_client.init()      # ← 초기화는 여기서
    ...
```

그런데 `redis_client` **객체 자체는 이미 import 때 만들어져 있습니다**.
즉 `lifespan`은 "만드는" 곳이 아니라 "이미 만들어진 전역에 시동 거는" 곳이 돼버렸습니다.
그래서 **import 순서가 곧 실행 순서**가 되고, 파일 위치 하나로 동작이 바뀌는 취약한 구조가 됩니다.

여기에 `@cache`(`core/conf.py:361`)와 `@lru_cache`(`plugin/core.py:48`)로 굳혀둔 전역 캐시까지 얹혀 있어, 런타임에 값을 바꿔도 반영되지 않습니다.

### 초보자용 요약: 왜 신경 써야 하나

| 하고 싶은 일 | 지금 왜 안 되나 |
| --- | --- |
| DB 없이 단위 테스트 | import만 해도 커넥션 풀이 생김 |
| 테스트용 DB로 바꿔치기 | `async_db_session`이 전역 상수라 교체 불가 |
| 설정 다르게 두 번째 인스턴스 | `settings`가 `@cache` 전역이라 불가능 |
| DB 죽었을 때 우아하게 재시도 | `sys.exit()`로 즉사 |
| 테스트마다 깨끗한 상태 | 전역 싱글턴이 이전 테스트 상태를 물고 있음 |

### 어떻게 고치나

**원칙: 만드는 건 `lifespan`, 보관은 `app.state`, 전달은 `Depends`.**

```python
# 1) 만들기 — lifespan 안에서
@asynccontextmanager
async def lifespan(app: FastAPI):
    engine = create_async_engine(get_database_url())
    redis = RedisCli()
    await redis.ping()                       # 실패하면 raise (sys.exit 금지)
    yield {'engine': engine, 'redis': redis} # ← app.state 에 담김
    await engine.dispose()
    await redis.aclose()

# 2) 꺼내 쓰기 — Depends 로
async def get_redis(request: Request) -> RedisCli:
    return request.app.state.redis

RedisDep = Annotated[RedisCli, Depends(get_redis)]

# 3) 서비스도 전역 인스턴스 대신 주입
async def get_user_service(db: CurrentSession) -> UserService:
    return UserService(db)

UserServiceDep = Annotated[UserService, Depends(get_user_service)]
```

이렇게 하면 테스트에서 한 줄로 통째로 갈아끼울 수 있습니다.

```python
app.dependency_overrides[get_redis] = lambda: FakeRedis()
```

이게 FastAPI가 `Depends`를 만든 이유이고, 지금 프로젝트가 못 쓰고 있는 기능입니다.
