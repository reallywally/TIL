# TypedDict를 알아보자

## TypedDict란?

파이썬 표준 라이브러리 typing 모듈에서 제공하는 기능으로, 딕셔너리 데이터 구조에 타입을 정의할 수 있게 해준다.

## 기본 사용법

```python
# 기본 딕셔너리
user1 = {
    "id": 1,
    "name": "Alice",
    "is_active": True
}

# ----------------------------
# TypedDict
from typing import TypedDict

class UserDict(TypedDict):
    id: int
    name: str
    is_active: bool

# 올바른 사용
user2: UserDict = {
    "id": 1,
    "name": "Alice",
    "is_active": True
}

# 타입 오류
user3: UserDict = {
    "id": "USR0002",  # id는 int이어야 하는데 문자값으로 들어옴!
    "name": "Bob",
    "is_active": True
}
```

## 언제 사용할까

명확하게 언제 사용해라고 정의하긴 어렵지만 **딕셔너리에 타입 힌트를 제공**하고 싶을 때 사용하며 아래와 같이 API 응답을 예시로 들 수 있다.

```python
class ApiResponse(TypedDict):
    status: str
    data: List[Product]
    message: Optional[str]

def get_product_names(response: ApiResponse) -> List[str]:
    """API 응답에서 상품 이름 목록을 추출하는 함수."""
    if response['status'] == 'success':
        return [product['name'] for product in response['data']]
    return []
```

## 그럼 Dataclass는 어떤 차이지?

얼핏 보면 Dataclass랑 동일하게 사용이 가능하다. TypedDict의 활용 예시를 찾아보면 위에 처럼 API 응답, Config 설정 등이 나왔는데 나는 항상 이거를 class로 만들어서 사용하고 있었기 때문에 실제 TypedDict 활용에 대해서는 공감이 잘 안됬다. 특히 딕셔너리는 속성에 접근할때 점(.)이 아닌 텍스트로 접근하여 오타가 발생하면 버그를 찾기 어려워 가능하면 class로 활용했다.  
하지만 다시 한번 강조하는 부분은 TypedDict의 목적은 **타입 힌트** 임을 잊지 말자. 상황에 따라 딕셔너리를 쓰지만 타입이 중요한 경우가 있기에 상황에 맞게 쓰도록 하자.

|구분|TypedDict|Dataclass|
|---|---|---|
|기반|dict|클래스 객체|
|접근 방식|user["id"]|user.id|
|런타임|제약 없음 (정적 검사만)|생성자에서 강제됨|
|가변성|기본적으로 가변(dict)|불변 설정 가능 (frozen=True)|
|용도|JSON, API 응답, 설정 등 dict 기반 데이터 구조|도메인 모델, 비즈니스 로직 객체|
