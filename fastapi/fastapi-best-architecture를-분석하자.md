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

## 자원 라이프사이클

## 정리
