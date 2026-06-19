# Item 12 객체를 출력할 때 repr과 str의 차이를 이해하라

## str — 사람을 위한 표현

print()는 내부적으로 str()을 호출한다. 그래서 결과는 사람이 읽기 좋게 나오지만 타입 정보가 숨겨집니다.

```python
# 이 여섯 가지는 모두 같은 출력
print("foo bar")
print(str("foo bar"))
print("%s" % "foo bar")
print(f"{'foo bar'}")
print(format("foo bar"))
print("foo bar".__str__())
# 전부 → foo bar  (따옴표 없음)
```

문제는 숫자 5와 문자열 '5'를 구분할 수 없다는 것이다.

```python
int_value = 5
str_value = "5"
print(int_value)                        # 5
print(str_value)                        # 5
print(f"Is {int_value} == {str_value}?")  # Is 5 == 5?
```

출력만 보면 타입이 다른지 전혀 알 수 없다.

## repr — 개발자를 위한 표현

repr()은 객체를 Python 코드로 재현 가능한 형태로 보여준다.

```python
a = "\x07"          # 벨 문자 (출력하면 아무것도 안 보임)
print(repr(a))      # '\x07'  ← 실제 내용이 보임
b = eval(repr(a))   # eval로 복원 가능!
assert a == b       # ✅
```

이는 디버깅할 때 타입 구분이 명확해짐:

```python
print(repr(int_value))  # 5    ← 숫자
print(repr(str_value))  # '5'  ← 따옴표가 있으므로 문자열임을 알 수 있음

# 포맷 문자열에서도 사용 가능:
print("Is %r == %r?" % (int_value, str_value))
print(f"Is {int_value!r} == {str_value!r}?")
# 모두 → Is 5 == '5'?  ← 타입 차이가 명확
```

## 사용자 정의 클래스에서의 repr/str

### 기본 repr

메모리 주소가 호출되어 쓸모가 없음

```python
class BetterClass:
    def __init__(self, x, y):
        self.x = x
        self.y = y

obj = BetterClass(1, "foo")
print(obj)
# → <__main__.BetterClass object at 0x1009be510>
# 내부 상태가 전혀 안 보임!
```

### __repr__ 구현

디버깅할때 아주 좋다.

```python
class BetterClass:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"BetterClass({self.x!r}, {self.y!r})"

obj = BetterClass(2, "bar")
print(obj)         # BetterClass(2, 'bar')
print(str(obj))    # BetterClass(2, 'bar')  ← __str__ 없으면 __repr__ 폴백
```

### __str__도 따로 구현

__repr__이 있더라도 사용자 표시용으로 __str__을 구현하기도 한다.

```python
class BetterClass:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return f"({self.x}, {self.y})"  # 간결한 사용자용 표현

    def __repr__(self):
        return f"BetterClass({self.x}, {self.y!r})"


obj2 = BetterClass(2, "bar")
print("Human readable:", obj2)  # Human readable: (2, bar)
print("Printable: ", repr(obj2))  # Printable: BetterClass(2, 'bar')

```

### 호출 우선순위 정리

- str(obj) 호출 시:
  1. __str__ 있으면 → 호출
  2. 없으면 → __repr__ 폴백
  3. 둘 다 없으면 → object.__repr__ (메모리 주소 형태)

- repr(obj) 호출 시:
  1. __repr__ 있으면 → 호출
  2. 없으면 → object.__repr__ (메모리 주소 형태)

## 매번 __repr__을 구현해야 되는가

__repr__이 디버깅에 좋다는건 이해하는데 생선하는 클래스마다 모두 __repr__을 구현하면 많이 번거로운 일이다.
그럴땐 ```@dataclass```를 쓰면 __repr__을 자동으로 만들어줘서 직접 작성할 필요가 없다.

## 한줄 요약

str로 객체 정보를 확인할 수 있지만 디버깅할때는 repr이 더 좋다.
