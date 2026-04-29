# __all__를 알아보자

## 이게 뭘까

간단하게 설명하면 ```from module import *``` 방식으로 임포트될 때 외부에 공개할 식별자들을 정의하는 리스트이다.

## 예시 코드

```python
# my_math.py
__all__ = ['add', 'subtract']  # 'multiply'는 목록에서 제외함

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def _internal_helper():
    print("이것은 내부용 함수입니다.")
```

```python
# main.py
from my_math import *

print(add(1, 2))       # 출력: 3
print(subtract(5, 2))  # 출력: 3

# print(multiply(2, 3)) 
# NameError: name 'multiply' is not defined 발생!
# __all__에 포함되지 않았기 때문에 import 되지 않음
```

## 주요 용도

```from <모듈> import *``` 구문을 사용하면 언더스코어로 시작하지 않는 모든 이름을 네임스페이스에 가져온다. 하지만 이렇게 전체를 가져오면 노출되지 않아도 되는 클래스, 함수들이 노출될 가능성이 있다. 그래서 불필요한 import를 방지하고 가독성을 위해서 사용한다.

## 주의사항

기본적으로 ```import *```는 파이썬 커뮤니티 자체에서 권장하지 않는다. ```__all__```를 잘 쓰는것도 중요하지만 에초에 **필요한것만 import하는 습관**을 들여야 한다.  
그리고 ```__all__```에 필요한것만 있다 한들 ```from my_math import multiply``` 처럼 개별 임포트를하면 정상적으로 가져온다.
