# item13 암묵적 문자열 연결보다 명시적 연결을 써라, 특히 리스트에서

## 암묵적 연결이란?

다른 언어들과 다르게 파이썬은 + 없이 문자열 리터럴이 나란히 있으면 자동으로 합쳐버린다.

```python
# 이 두 코드는 완전히 동일
my_test1 = "hello" "world"
my_test2 = "hello" + "world"
assert my_test1 == my_test2   # ✅ "helloworld"
```

## 유용한 경우

제목에서 암묵적인거 보다 명시적으로 쓰하고 했지만 그래도 암묵적인 상황이 유용한 경우가 있다. 아래와 같이 긴 문자열을 여러 줄로 나눌 때, 서로 다른 종류의 문자열 리터럴을 섞어야 할 때 편하다.

```python
x = 1
my_test = (
    r"first \ part is here with escapes\n, "   # raw string
    f"string interpolation {x} in here, "      # f-string
    'this has "double quotes" inside'           # single-quoted
)
```

## 위험한 경우

### 함정 1: 쉼표(,) 실수로 암묵적 연결 발생

만약 3개 요소짜리 리스트를 만들려고 했는데 쉼표를 빼먹고 오타가 발생하면 의도와 전혀 다른 결과를 볼 수 있다. 이런 경우 가독성도 떨어저서 버그를 찾기가 힘들다.

```python
my_test = r"fir\st", f"{y}" '"third"'
print(my_test)
# → ('fir\\st', '2"third"')   ← 2개 요소! f-string과 일반 문자열이 합쳐짐
```

### 함정 2: 쉼표 실수로 삭제되면 요소가 합쳐짐

함정 1과 비슷한 경우로 쉼표가 누락되면서 의도치 않은 리스트가 생성될 수 있다.

```python
# 의도: 3개 요소 리스트
my_test1 = [
    "first line\n",
    "second line\n",
    "third line\n",
]
# → ['first line\n', 'second line\n', 'third line\n']  ✅ 정상

# 중간 쉼표 실수로 삭제
my_test2 = [
    "first line\n",
    "second line\n"   # ← 쉼표 빠짐!
    "third line\n",
]
print(my_test2)
# → ['first line\n', 'second line\nthird line\n']  ❌ 2개 요소로 합쳐짐!
```

그나마 다행이라면 auto-formatter가 이걸 발견하면 재배치해서 힌트를 주거나 해결준다.

```python
# black은 이렇게 분리해서 주고
my_test1 = [
    "first line\n",
    "second line\n" "third line\n",   # 같은 줄로 합쳐줌 → 의심스러워 보임
]

# ruff는 통합해 버린다
my_test2 = ["a", "b" "c"] # -> ["a", "bc"]
```

## 해결책

제목 그대로 가능하면 + 연산자를 사용하자.

```python
my_test = [
    "first line\n",
    "second line\n" +   # + 가 있으면 의도가 명확!
    "third line\n",
]
```

## 한줄 요약

+가 없으면 자동으로 합쳐지고 의도치 않은 버그가 발생하니 명시적으로 +를 사용하자.
