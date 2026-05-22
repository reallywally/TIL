# Item12 객체를 출력할 때 repr과 str의 차이를 이해하라

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
