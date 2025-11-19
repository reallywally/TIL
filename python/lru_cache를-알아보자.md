# lru_cach를 알아보자

## lru_cach가 뭔데?

lru_cach는 함수 **결과를 자동으로 캐싱**해주는 데코레이터이다. LRU(Least Recently Used) 방식으로 캐시를 관리하기 때문에 최근에 사용된 값은 유지되고 오래 안 쓰인 값은 자동으로 제거된다. 그리고 내부 동작 방식을 초초간단하게 설명하면 내부적으로는 **OrderedDict 기반의 효율적 자료구조**를 활용한다.

## 기본 사용 예제

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def slow_function(x):
    print("실행됨:", x)
    return x * 2

print(slow_function(10))  # 실행됨 출력 + 결과
print(slow_function(10))  # 캐싱된 결과, 실행됨 출력 없음
```

두 번째 호출에서 slow_function(10)은 실제 함수를 실행하지 않는다. 메모리에서 바로 꺼내기 때문에 속도가 압도적으로 빠르다.

## 언제 사용하면 좋을까?

결과값이 캐싱되서 좋다는건 이해가 된다. 그럼 언제 사용해야 좋을까? 

### 1. 비용이 큰 연산 함수

복잡한 수학 계산, 대량 데이터 전처리, 머신러닝 피처 계산 등 한번 실행할때 많은 리소스가 캐싱이 필요한 대표적인 예시이다.

### 2. 외부 API / DB 조회 결과가 자주 반복될 때

```python
@lru_cache
def get_country_info(code):
    return requests.get(f"https://api.example.com/country/{code}").json()
```
