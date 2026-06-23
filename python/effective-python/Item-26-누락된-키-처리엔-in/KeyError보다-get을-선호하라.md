# Item 26 누락된 키 처리엔 in/KeyError보다 get을 선호하라

딕셔너리의 기본 연산은 접근, 대입, 삭제 세 가지인데, 내용이 동적이라 없는 키에 접근하거나 삭제하려는 상황이 흔히 생긴다. 이걸 어떻게 처리하느냐가 이 항목의 주제이다.

## 접근

빵 종류별 투표 수를 세는 카운터를 예로 데이터를 꺼내오는(접근)하는 방법을 알아보자

```python
counters = {"pumpernickel": 2, "sourdough": 1}
key = "wheat"
```

### 방법 1: in 연산자

키 접근 2번 + 대입 1번이 필요.

```python
if key in counters:
    count = counters[key]
else:
    count = 0
counters[key] = count + 1
```

### 방법 2: KeyError 예외

이론상 접근 1번 + 대입 1번으로 더 효율적(예외 발생·포착 비용은 논외).

```python
try:
    count = counters[key]
except KeyError:
    count = 0
counters[key] = count + 1
```

### 방법 3 (권장): get 메서드

두 번째 인자가 키가 없을 때의 기본값. 접근 1번 + 대입 1번인데 훨씬 짧고 예외 처리 오버헤드도 없음.

```python
count = counters.get(key, 0)
counters[key] = count + 1
```

in이나 KeyError를 더 짧게 줄이는 변형들(if key not in ..., counters[key] += 1 등)도 있지만, 전부 대입 코드가 중복되어 가독성이 떨어지므로 피하는 게 좋다. 단순 타입 값에는 get이 가장 짧고 명확하다.