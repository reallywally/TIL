# item18 여러 이터레이터를 병렬 처리할 땐 zip을 써라

## 문제 상황

서로 관련된 여러 리스트를 다룰 일이 많다. 예를 들어 이름 리스트와, 각 이름의 글자 수 리스트가 있다고 해보자

```python
names = ["Cecilia", "Lise", "Marie"]
counts = [7, 4, 5]
```

두 리스트의 항목은 **같은 위치(인덱스)** 로 서로 연결된다. 그래서 두 리스트를 한 반복문에서 같이 쓰려면, range(len(...))로 인덱스를 만들어 양쪽에 인덱싱하는 방식을 기본으로 떠올리게 된다.

```python
longest_name = None
max_count = 0
for i in range(len(names)):
    count = counts[i]
    if count > max_count:
        longest_name = names[i]
        max_count = count
```

문제는 names[i], counts[i] 같은 인덱싱이 시각적으로 지저분하고, 같은 i로 두 배열을 인덱싱하는 게 코드도 길고 가독성도 좋지 않다.

## 해결책: zip

파이썬의 zip은 둘 이상의 이터레이터를 게으른(lazy) 제너레이터로 감싸서, 각 이터레이터의 다음 값을 묶은 튜플을 차례로 내놓는다. 이 튜플을 for 문에서 바로 언패킹하면 인덱싱이 사라져 훨씬 깔끔해진다.

```python
longest_name = None
max_count = 0
for name, count in zip(names, counts):
    if count > max_count:
        longest_name = name
        max_count = count
```

zip은 감싼 이터레이터를 한 번에 한 항목씩 소비하여 무한히 긴 입력에도 메모리 폭주 없이 쓸 수 있다.

## 길이가 다를 때를 조심하라

입력 이터레이터의 길이가 서로 다르면 주의해야 한다. 예를 들어 names에만 항목을 추가하고 counts 갱신을 깜빡하면:

```python
names.append("Rosalind")
"""
names = ["Cecilia", "Lise", "Marie", "Rosalind"]
counts = [7, 4, 5]
"""

for name, count in zip(names, counts):
    print(name)
# Cecilia / Lise / Marie  ← "Rosalind"는 출력 안 됨!
```

zip은 감싼 이터레이터 중 하나라도 소진되면 멈춘다. 즉 출력 길이는 가장 짧은 입력에 맞춰지고, 이 잘림은 조용히(silently) 일어난다.

## 조기 잘림 해결

조기 잘림 문제는 크게 3가지로 해결할 수 있다.

### 1. ruff 같은 포맷터

포맷터를 이용하여 경고를 확인하고 길이를 미리 맞추면 된다.

### 2. strict=True

zip의 파라미터로 strict=True를 넘기고 길이가 안 맞을때 예외가 발생하도록 한다.

```python
for name, count in zip(names, counts, strict=True):
    print(name)
# ... ValueError: zip() argument 2 is shorter than argument 1
```

### 3. 기본값 설정

반대로 짧은 쪽을 기본값으로 채우고 싶다면 itertools.zip_longest를 쓰면된다.

```python
from itertools import zip_longest
for name, count in zip_longest(names, counts, fillvalue=0):
    print(name, count)
```

## 자주 쓰이는 패턴 / 실무 예쩨

### 1. 병렬 반복

```python
for x, y in zip(list1, list2):
    # list1과 list2를 같이 반복
```

### 2. 행렬 전치

```python
matrix = [[1,2,3], [4,5,6], [7,8,9]]
cols = list(zip(*matrix))    # [(1,4,7),(2,5,8),(3,6,9)]
```

### 3. 헤더와 데이터를 행으로 합쳐서 딕셔너리 만들기

```python
headers = ["name", "age", "city"]
data = ["alice", 30, "서울"]

d = dict(zip(headers, data))  # {'name':'Alice','age':30,'city':'Seoul'}
```

## 한줄 요약

zip은 여러 이터레이터를 병렬로 순회할 때 쓴다.
