# TIL: Spring의 DTO 파일 분리 vs FastAPI의 schemas.py 통합

Spring에서는 Request/Response DTO를 클래스 파일로 하나하나 분리하는데, FastAPI에서는 `schemas.py` 한 파일에 다 몰아넣는다. 개인적으로는 하나씩 분리하는게 더 유지보수가 좋은거 같은데 파이썬 백엔드 에서는 왜 통합하는지 궁금해젔다.

## 결론부터

단순한 관습 차이가 아니라 **언어 철학 + 타입 시스템 + 프레임워크 성향**이 겹친 결과다. Spring의 파일 분리는 "가치관"이라기보다 **언어가 강제한 것**이고, FastAPI의 통합은 **Python이 허용한 자유를 활용한 것**이다. 출발점 자체가 다르다.

## 1. 언어 차원: 파일 분리 강제 vs 자유

Java는 `public` 클래스는 **파일명과 같아야 하고, 파일당 하나**라는 규칙이 언어 레벨에 박혀 있다. DTO가 10개면 파일도 10개가 강제된다. 선택의 여지가 없다.

Python에는 그런 제약이 없다. 한 모듈(`.py`)에 클래스를 몇 개 넣든 자유다. 그래서 "관련된 것끼리 한 파일에 모은다"는 게 자연스러운 선택지가 된다.

## 2. 모듈 단위의 차이

Java/Spring은 **패키지 + 클래스**가 모듈 경계다. 그래서 응집도를 디렉터리 구조로 표현한다. `dto/request/`, `dto/response/` 같은 폴더로 조직화한다.

Python은 **`.py` 파일 자체가 모듈(네임스페이스)** 다. `schemas.py` 하나가 이미 하나의 응집 단위다.

```python
from app.schemas import UserCreate, UserResponse
```

파일이 네임스페이스 역할을 한다. Spring에서 `dto` 패키지에 해당하는 게 Python에서는 `schemas.py` 파일 하나인 셈이다. **추상화 레벨이 한 단계씩 밀려 있는 것**이지, 조직화를 안 하는 게 아니다.

## 3. 보일러플레이트 비용의 차이

Spring DTO는 필드, getter/setter, 생성자, 검증 애노테이션 등으로 클래스 하나가 수십 줄이 된다. Lombok을 써도 애노테이션이 붙는다. 파일당 하나여도 각 파일이 충분히 무겁다.

반면 Pydantic 모델은 이렇게 짧다.

```python
class UserCreate(BaseModel):
    email: EmailStr
    password: str

class UserResponse(BaseModel):
    id: int
    email: EmailStr
```

10줄짜리 클래스를 파일 10개로 쪼개면 오히려 파일 탐색 오버헤드만 커진다. **선언이 가벼우니 한곳에 모으는 게 이득**이다.

## 4. 밑바탕 가치관

Spring 진영은 **명시적 구조화, 계층 분리, 엔터프라이즈 규모에서의 예측 가능성**을 중시한다. 대규모 팀과 코드베이스에서 파일 위치만 봐도 역할을 알 수 있게 하는 것이다.

FastAPI/Python 진영은 **실용성, 최소한의 의식(ceremony), 응집도(locality)**를 중시한다. "관련된 스키마는 같이 있어야 읽기 쉽다"는 쪽이다.

## 실무 팁: schemas.py는 기본값일 뿐이다

`schemas.py` 한 파일은 **작은~중간 프로젝트의 기본값**일 뿐이다. 규모가 커지면 Python에서도 패키지로 쪼개는 게 정석이다.

```text
app/
  schemas/
    __init__.py      # 외부 노출용 re-export
    user.py          # UserCreate, UserUpdate, UserResponse
    order.py
    common.py        # 공통 페이지네이션 등
```

이때 **도메인 단위(user, order)로 나누는 것**이 요청/응답 단위로 나누는 것보다 응집도가 높다. Spring의 `dto/request`, `dto/response` 식 "기술적 역할별 분리"는 Python 커뮤니티에서는 오히려 안티패턴에 가깝게 본다. 한 도메인을 고치려고 파일 여러 개를 오가야 하기 때문이다.

## 한 줄 요약

Spring은 **언어가 파일 분리를 강제**하고 엔터프라이즈 예측 가능성을 중시한다. FastAPI는 **Python의 모듈 자유도 + 가벼운 Pydantic 선언** 덕에 응집도를 우선한다. 규모가 커지면 FastAPI도 도메인별 패키지로 나누는 게 정답이다.
