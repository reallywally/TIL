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
    "id": "USR0002",
    "name": "Bob",
    "is_active": True
}
```

## 언제 사용할까

## 그럼 dataclass는 어떤 차이지?
