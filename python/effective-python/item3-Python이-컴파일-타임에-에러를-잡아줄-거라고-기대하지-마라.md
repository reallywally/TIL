# Item3 Python이 컴파일 타임에 에러를 잡아줄 거라고 기대하지 마라

## Python이 실행 전에 잡아주는 것

기본적인 문법이 완전히 틀린 경우는 잘 잡아준다.

```python
if True  # 콜론 없음
    print('hello')
# → SyntaxError: expected ':'

1.3j5  # 잘못된 숫자 리터럴
# → SyntaxError: invalid imaginary literal
```

## Python이 실행 전에 못 잡는 것들 (함정들)

### 1. 변수 참조 오류 — 실행해야 비로소 발견됨

함수 정의할 때는 멀쩡해 보이지만 호출할 때 터짐.

```python
def bad_reference():
    print(my_var)   # my_var를 먼저 쓰고
    my_var = 123    # 그 다음에 정의

bad_reference() # → UnboundLocalError: cannot access local variable
```

### 2. 조건에 따라 터지기도 하고 안 터지기도 함

이게 가장 무서운 케이스이다. 물론 java도 특정 케이스에만 에러가 발생되는 코드가 만들어 질 수 있다. 하지만 아래 코드처럼 java에서는 아에 작성도 못할 코드를 python에서는 일단 작성이 가능하기 때문에 비즈니스 로직이 없는 단순한 코드에서도 발생할 가능성이 매우 높다.

```python
# case1
def sometimes_ok(x):
    if x:
        my_var = 123
    print(my_var)

sometimes_ok(True)   # → 123  (정상)
sometimes_ok(False)  # → UnboundLocalError  (폭발)

# case2
def bad_math():
    return 1 / 0

bad_math()  # → ZeroDivisionError: division by zero
```

### 3.  그 외에도

존재하지 않는 메서드 호출, 잘못된 인자 개수, 타입 불일치 등 모두 실행 전엔 통과된다.

## 왜 이럴까?

python은 언어 자체가 **런타임의 유연성을 컴파일 타임의 에러 검출보다 우선시**하도록 설계 되었다.

## 어떻게 해결할까?

다행히 발전된 현대 문명으로 이런 에러를 크게 줄일 수 있다. 대표적으로 flake8, pylint, ruff같은 린터로 실행 전에 명백한 실수 잡을 수 있다. 그라고 mypy, pyright타입 체커. 타입 힌트 기반으로 더 깊은 분석도 가능하다.
