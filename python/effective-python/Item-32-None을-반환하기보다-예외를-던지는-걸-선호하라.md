# Item 32 None을 반환하기보다 예외를 던지는 걸 선호하라

## 흔한 유혹: 특수 상황에 None 반환

유틸리티 함수를 짤 때 반환값 None에 특별한 의미를 부여하고 싶어진다. 예를 들어 나눗셈 헬퍼에서 0으로 나누는 경우, 결과가 정의되지 않으니 None을 돌려주는 게 자연스럽고 예외처리도 잘 된것 처럼 보인다.

```python
def careful_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return None

result = careful_divide(1, 0)
if result is None:
    print("잘못된 입력")
```

## 문제: falsey 값과 헷갈린다

careful_divide에서 분자가 0이면 어떻게 될까? 분모가 0이 아니면 결과로 0(정상 값)을 반환한다. 문제는 이 0을 if 조건에서 평가할 때 생긴다. None만 검사해야 하는데, 실수로 모든 falsey 값을 에러로 취급하기 쉽다.

```python
x, y = 0, 5
result = careful_divide(x, y)
if not result:              # 0도 falsey라 여기 걸림!
    print("잘못된 입력")     # 실행됨 — 하지만 그러면 안 됨!
```

0, 빈 문자열 "", 빈 리스트 등도 전부 불리언 평가에서 False가 되기 때문에, None에 특별한 의미를 줄 때 흔히 저지르는 실수이다.

## 개선 방법

### 방법1. 튜플 반환 (그러나 불완전)

(성공 여부, 실제 결과) 튜플로 나눠 반환하는 방법이다.

```python
def careful_divide(a, b):
    try:
        return True, a / b
    except ZeroDivisionError:
        return False, None

success, result = careful_divide(x, y)
if not success:
    print("잘못된 입력")
```

호출자가 상태 부분을 보게 강제되긴 하지만, 첫 번째 값을 _로 무시해버리면 결국 None 반환과 똑같은 함정에 빠진다.

```python
_, result = careful_divide(x, y)
if not result:              # 다시 원점 — 0이 걸림
    print("잘못된 입력")
```

### 방법2 (권장): 예외를 던져라

특수 상황에 None을 반환하지 말고, 예외를 호출자에게 던져 처리하게 하는 게 낫다. 여기선 ZeroDivisionError를 잡아 "입력이 잘못됐다"는 의미의 ValueError로 바꿔 던진다.

```python
def careful_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        raise ValueError("잘못된 입력")
```

이제 호출자는 반환값을 조건 검사할 필요가 없다. 반환값은 항상 유효하다고 가정하고, try 뒤의 else 블록에서 바로 사용하면 된다.

```python
x, y = 5, 2
try:
    result = careful_divide(x, y)
except ValueError:
    print("잘못된 입력")
else:
    print(f"결과: {result:.1f}")   # 결과: 2.5
```

### 타입 애너테이션 + docstring으로 마무리

이 방식은 타입 힌트와 잘 어울린다. 반환 타입을 float로 명시하면 절대 None이 아님을 표현할 수 있다. 다만 파이썬의 점진적 타이핑은 (자바의 checked exception처럼) "어떤 예외를 던지는지"를 타입으로 표현하는 수단은 일부러 제공하지 않는다. 그래서 예외 동작은 docstring에 문서화하고, 호출자가 그걸 보고 어떤 예외를 잡을지 알게 해야 한다.

```python
def careful_divide(a: float, b: float) -> float:
    """a를 b로 나눈다.

    Raises:
        ValueError: 입력을 나눌 수 없을 때.
    """
    try:
        return a / b
    except ZeroDivisionError:
        raise ValueError("잘못된 입력")
```

이러면 입력, 출력, 예외 동작이 모두 명확해지고, mypy --strict도 통과하며, 호출자가 실수할 가능성이 매우 낮아진다.

## 한줄 요역

특수 상황을 나타낼 땐 None 대신 예외를 던지자.
