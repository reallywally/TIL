# Item 35 키워드 인자로 선택적 동작을 제공하라

## 위치 인자 vs. 키워드 인자

대부분 언어처럼 파이썬도 함수 호출 시 **위치(position)**로 인자(parameter)를 넘길 수 있다.

```python
def remainder(number, divisor):
    return number % divisor

assert remainder(20, 7) == 6
```

일반 인자는 전부 **키워드(keyword)**로도 넘길 수 있다. 필수 위치 인자만 다 채우면, 키워드 인자는 순서 무관하게 섞어 쓸 수 있다. 아래는 모두 동일하게 동작한다.

```python
remainder(20, 7)
remainder(20, divisor=7)
remainder(number=20, divisor=7)
remainder(divisor=7, number=20)
```

단, 위치 인자는 키워드 인자보다 먼저 와야 하며 각 인자는 한 번만 지정할 수 있다.

## 딕셔너리를 **로 펼치기

이미 딕셔너리가 있다면 **연산자로 키-값 쌍을 키워드 인자로 펼쳐 넘길 수 있다.

```python
my_kwargs = {"number": 20, "divisor": 7}
assert remainder(**my_kwargs) == 6
```

## 키워드 인자의 세 가지 이점

### 호출 코드가 명확해진다
