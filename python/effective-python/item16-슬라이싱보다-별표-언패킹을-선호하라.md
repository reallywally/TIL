# item16 슬라이싱보다 별표(*) 언패킹을 선호하라

## 문제 상황

기본 언패킹은 시퀀스의 길이를 미리 알아야 한다는 한계가 있다. 그리고 길이가 안 맞으면 런타임 에러가 발생한다.

```python
ages = [0, 9, 4, 8, 7, 20, 19, 1, 6, 15]
descending = sorted(ages, reverse=True)
oldest, second_oldest = descending   # ValueError: too many values to unpack
```

이럴 때 아래와 같이 인덱싱과 슬라이싱에 의존하는 경우도 많다.

```python
oldest = descending[0]
second_oldest = descending[1]
others = descending[2:]
```

이것도 동작은 하지만, 가독성이 떨어지고 list 구조에서 인덱스로 직접 접근할 경우 예상치 못한 버그가 발생할 가능성이 높다.

## 해결책: 별표 표현식 (catch-all unpacking)

파이썬은 패턴의 다른 부분에 매칭되지 않은 나머지 값을 한 변수가 전부 받아주는 방식인 **별표(*) 표현식**을 지원한다.

```python
oldest, second_oldest, *others = descending
# others에 나머지가 전부 리스트로 들어감
```

훨씬 짧고, 가독성이 좋으며, 잘못된 인덱스 접근을 막을 수 있다.  

별표는 처음, 중간, 끝 어디든 어느 위치에나 올 수 있다

```python
oldest, *others, youngest = descending          # 처음·끝만 뽑고 중간은 others
*others, second_youngest, youngest = descending  # 끝 두 개만 따로
```

## 주의할 규칙들

별표 언팩킹은 좋아 보이지만 주의해야할 사항들이 있다.

### 1. 별표만 단독으로는 쓸 수 없다

별표 언팩킹은 최소 하나의 일반 변수가 필요하다.

```python
*others = descending → SyntaxError
```

### 2. 한 패턴에 별표 두 개는 안 된다

말 그대로 별표를 2개 이상 사용할 수 없다. 단, 중첩 구조의 서로 다른 레벨이라면 별표를 여러 번 쓸 수 있다(권장하진 않음).

```python
first, *mid, *mid2, last = [...] → SyntaxError
```

### 3. 별표 변수는 항상 리스트가 된다

그리고 남는 값이 없으면 []와 같이 빈 리스트로 저장이 된다.

```python
short = [1, 2]
first, second, *rest = short
print(first, second, rest)   # 1 2 []
```

## 이터레이터에 특히 유용

앞서 item5에도 언급한 내용을 다시보면 언팩킹할때 속성이 많을시 dataclass, NamedTuple, BaseModel 등 class로 처리하는게 좋다고 했다. 하지만 정말 앞에 일부분만 써야할때는 class로 만들기가 번거롭고 비효율적이다. 이럴때 별표와 함께 쓰면 좋다.  
대표적인 상황을 예로 들면 CSV 제너레이터에서 헤더와 나머지 행을 분리할 때 깔끔합니다.

```python
it = generate_csv()
header, *rows = it
print("헤더:", header)
print("행 수:", len(rows))
```

## 한줄 요약

별표 언팩킹을 하면 언팩킹시 나머지 데이터를 쉽게 정리할 수 있다.
