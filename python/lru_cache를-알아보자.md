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

## 주의해야 할 점

### 1. 인자는 반드시 해시 가능 해야 한다

즉, list 같은 mutable 타입은 사용할 수 없다. 튜플이나 string 같은 immutable 타입으로 변환해서 쓰자.

```python
@lru_cache
def test(a):   # a가 list면 Error
    return a
```

### 2. 메모리 관리

캐시되는 maxsize를 너무 크게 잡지 말고 적정 수치를 정해야 한다. 서버 스펙과 사용 환경에 따라 달라지겠지만 GPT씨는 128 또는 1024 정도가 적절하다 한다.

### 3. 캐시 초기화

배포, 데이터 구조 변경, 환경 초기화 등 상황에 따라 적정 시점에 캐시를 정리하는것이 좋다.

```python
@lru_cache
def test(a):  
    return a

test.cache_clear()  # 캐시 초기화
```

## 기타

```lur_cach```의 성능 개선 예시로 피보나치 재귀 함수를 제일 많이 언급한다. 그런데 매번 다른 값을 리턴하는데 피보나치가 어떻게 lur_cach의 대표적인 예시로 나오는지 잘 이해되지 않았다. GPT씨에게 물어보니 그림으로 정리해주었고 한번에 이해가 되었다.

```text
fibonacci(10)
 ├─ fibonacci(9)
 │   ├─ fibonacci(8)
 │   └─ fibonacci(7)
 └─ fibonacci(8)
     ├─ fibonacci(7)
     └─ fibonacci(6)
```

이 그림대로 하면 중복 계산이 한눈에 보인다. fibonacci(8)은 fibonacci(10) 계산 과정에서 최소 2번 호출된다. 실제로는 더 깊은 레벨로 내려가면서 동일한 호출이 수십, 수백 번 발생한다.
