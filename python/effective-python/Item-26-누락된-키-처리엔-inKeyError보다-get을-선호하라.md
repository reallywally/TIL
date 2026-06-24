# Item 26 누락된 키 처리엔 in/KeyError보다 get을 선호하라

딕셔너리의 기본 연산은 접근, 대입, 삭제 세 가지인데, 내용이 동적이라 없는 키에 접근하거나 삭제하려는 상황이 흔히 생긴다. 이걸 어떻게 처리하느냐가 이 항목의 주제이다.

## 접근

빵 종류별 투표 수를 세는 카운터를 예로 데이터를 꺼내오는(접근)하는 방법을 알아보자

```python
counters = {"pumpernickel": 2, "sourdough": 1}
key = "wheat"
```

### 방법 1: in 연산자

에러 방지를 위해 키 접근 2번 + 대입 1번이 필요하다.

```python
if key in counters:
    count = counters[key]
else:
    count = 0
counters[key] = count + 1
```

### 방법 2: KeyError 예외

예외 발생·포착 비용은 논외했을때 이론상 접근 1번 + 대입 1번으로 더 효율적이다.

```python
try:
    count = counters[key]
except KeyError:
    count = 0
counters[key] = count + 1
```

### 방법 3 (권장): get 메서드

두 번째 인자가 키가 없을 때의 기본값. 접근 1번 + 대입 1번인데 훨씬 짧고 예외 처리 오버헤드도 없다.

```python
count = counters.get(key, 0)
counters[key] = count + 1
```

in이나 KeyError를 더 짧게 줄이는 변형들(if key not in ..., counters[key] += 1 등)도 있지만, 전부 대입 코드가 중복되어 가독성이 떨어지므로 피하는 게 좋다.
단순 타입 값에는 get이 가장 짧고 명확하다.

## 값이 복잡한 타입(리스트)일 때

이번엔 "누가 어디에 투표했는지" 이름 리스트를 값으로 예시를 보자

```python
votes = {"baguette": ["Bob", "Alice"], "ciabatta": ["Coco", "Deb"]}
key = "brioche"
who = "Elmer"
```

### in 연산자

```python
if key in votes:
    names = votes[key]
else:
    votes[key] = names = []   # 3중 대입으로 한 줄에 처리
names.append(who)
```

### KeyError

```python
try:
    names = votes[key]
except KeyError:
    votes[key] = names = []
names.append(who)
```

### get

```python
names = votes.get(key)
if names is None:
    votes[key] = names = []
names.append(who)
```

### get + 대입 표현식(:=, Python 3.8+)

get 접근에 대입 표현식까지 사용하면 한 줄 더 짧고 가독성도 좋다.

```python
if (names := votes.get(key)) is None:
    votes[key] = names = []
names.append(who)
```

### setdefault — 더 짧지만 함정이 있다

dict에는 setdefault가 있다. 키가 있으면 그 값을, 없으면 기본값을 딕셔너리에 넣고 그 값을 반환한다.

```python
names = votes.setdefault(key, [])
names.append(who)
```

대입 표현식 버전보다 코드는 더 간결해 보이지만 두 가지 문제가 있다.

#### 1. 이름이 직관적이지 않다

값을 가져오는(get) 동작인데 왜 set일까? get_or_set 같은 이름이었으면 좋았을 텐데, 파이썬에 익숙하지 않은 사람은 의도를 파악하기 어렵다.

#### 2. 기본값이 복사되지 않고 그대로 딕셔너리에 들어간다 (중요한 함정)

```python
data = {}
value = []
data.setdefault("foo", value)
print("Before:", data)   # {'foo': []}
value.append("hello")    # 외부 value 변경
print("After: ", data)   # {'foo': ['hello']}  ← 딕셔너리를 변경한건 아닌데 반영됨!
```

그래서 setdefault를 쓸 땐 **매번 새 기본값을 만들어 넘겨야 한다**. 효율을 위해 객체를 재사용하면 이상한 버그가 생긴다. 결국 호출마다 리스트를 새로 할당하는 성능 부담이 있다.

## 한줄 요약

딕셔너리에서 값을 접근할때는 get을 사용하고 setdefault는 쓰지 말자
