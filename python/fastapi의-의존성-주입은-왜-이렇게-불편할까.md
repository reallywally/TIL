# fastAPI의 의존성 주입은 왜 이렇게 불편할까

FastAPI를 쓰다 보면 “분명 DI(Dependency Injection)인데… 왜 이렇게 매번 Depends(...)를 써야 하지?”, “스프링처럼 깔끔하게 ‘주입’되는 느낌이 아닌데?” 같은 찝찝함이 남는다.
이 글은 그 불편함이 단순히 프레임워크 완성도 차이가 아니라, 언어/런타임/철학 차이에서 오는 설계 결과라는 점을 정리해보려는 글이다.

## 스프링에서 의존성 주입 방법

스프링에서 DI는 보통 “컨테이너가 객체를 만들고(생명주기 관리), 필요한 의존성을 알아서 꽂아준다”는 경험으로 체감된다.

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class OrderService {
  private final PaymentClient paymentClient;

  public void test(){
    paymentClient.ping();

    log.info("야호!");
  }
}
```

여기서 핵심은 다음과 같다.  

- 스프링은 IoC 컨테이너가 “누가 무엇을 만들고, 언제 어떻게 재사용할지”를 책임진다.
- @Component, @Service, @Autowired 같은 메타데이터(어노테이션)를 기반으로 스캔 → 등록 → 그래프 구성 → 주입이 이루어진다.
- 개발자는 “이 클래스는 이런 역할이고, 이 의존성이 필요해”만 선언하면, 나머지를 컨테이너가 처리한다.

## FastAPI에서 의존성 주입 방법

FastAPI의 DI는 전형적으로 이렇게 보인다

```python
from fastapi import Depends, FastAPI
app = FastAPI()
def get_db():
    ...
@app.get("/items")
def read_items(db=Depends(get_db)):
    ...
```

혹은 Annotated로 더 “선언적으로” 만들기도 한다:

```python
from typing import Annotated
from fastapi import Depends

DB = Annotated[object, Depends(get_db)]

@app.get("/items")
def read_items(db: DB):
    ...
```

그리고 의존성은 엔드포인트뿐 아니라 라우터 단위 특정 경로 그룹, 보안(인증/인가), request-scoped 리소스(DB 세션 등) 같은 형태로도 걸 수 있다. 여기서 체감되는 포인트는 다음과 같다.

1. “컨테이너가 주입해주는” 감각보다는, **요청 처리 파이프라인에서 필요한 값을 ‘명시적으로 선언’**하는 감각이 강하다.
2. 특히 엔드포인트 시그니처에 Depends(...)가 계속 등장하니, 스프링에서 넘어오면 장황하고 투박하게 느껴질 수 있다.

## 왜 이렇게 다를까

결론부터 말하면, 둘은 DI라는 단어를 공유하지만 “DI를 어디에 최적화했는지”가 다르다.

- 스프링: 애플리케이션 내부 객체 그래프 구성/생명주기 관리에 최적화
- FastAPI: HTTP 요청 처리(입력 파싱/검증/주입) 파이프라인 구성에 최적화

FastAPI는 “전역 컨테이너가 전부를 관리”하기보다는, “요청이 들어왔을 때 이 핸들러를 실행하는데 필요한 값들을 어떤 순서로 준비할까?”에 초점이 있다. 그래서 의존성 주입이 request-time에 해석되는 선언에 가깝다.

또한 언어 차이도 크다.

- Java/Spring: 컴파일 타입 시스템 + 리플렉션 + 어노테이션 메타데이터를 강하게 활용하는 문화
- Python/FastAPI: 런타임 동적 언어 + 함수 시그니처/타입 힌트 기반의 “명시적 선언”을 선호하는 문화

이 차이가 “왜 FastAPI는 스프링처럼 안 해?”라는 질문의 대부분을 설명한다.

- 왜 어노테이션 처리가 안될까
- 가치관이 다른가
