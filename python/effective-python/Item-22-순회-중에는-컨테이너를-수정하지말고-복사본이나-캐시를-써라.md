# Item 22 순회 중에는 컨테이너를 수정하지말고 복사본이나 캐시를 써라

## 문제: 순회하면서 컨테이너를 바꾸면 동작이 들쭉날쭉

파이썬에는 순회 동작과 관련된 함정이 많다.(앞의 Item 21도 그중 하나).
가장 헷갈리는 건, 순회 중 컨테이너를 수정할 때 어떤 경우는 에러가 나고 어떤 경우는 멀쩡하다는 일관성 없는 동작한다는것이다.

### 딕셔너리 — 키 추가/삭제는 에러

아래 코드와 같이 새로운 순회 중 딕셔너리에 새로운 키로 아이템을 등록하면 에러가 발생한다.

```python
my_dict = {"red": 1, "blue": 2, "green": 3}
for key in my_dict:
    if key == "blue":
        my_dict["yellow"] = 4   # RuntimeError: dictionary changed size during iteration
```

키를 del로 지워도 같은 에러가 납니다. 그런데 기존 키의 "값"만 바꾸는 건 정상작동한다.
동일하게 딕셔너리를 변경하는거지만 추가는 안되고 수정은 되는게 일관성이 없어 헷갈린다.

```python
for key in my_dict:
    if key == "blue":
        my_dict["green"] = 4    # 기존 키의 값만 변경 → 괜찮음
```

### 셋(set) — 크기를 바꾸면 에러

set도 기본적으로는 순회 중 값을 추가하면 에러가 발생한다.

```python
my_set = {"red", "blue", "green"}
for color in my_set:
    if color == "blue":
        my_set.add("yellow")    # RuntimeError: Set changed size during iteration
```

하지만 이미 있는 원소를 다시 추가하는 건 크기가 안 변하므로 괜찮다.

```python
my_set = {"red", "blue", "green"}
for color in my_set:
    if color == "blue":
        my_set.add("green")
```

### 리스트 — 더 어려움

인덱스를 이용하여 기존 값을 덮어쓰는 건 괜찮다.

```python
my_list = [1, 2, 3]
for number in my_list:
    if number == 2:
        my_list[0] = -1   # 괜찮음
```

그런데 현재 순회 위치 앞에 삽입(insert)하면 무한 루프에 빠진다.

```python
my_list = [1, 2, 3]
for number in my_list:
    print(number)
    if number == 2:
        my_list.insert(0, 4)   # 1, 2, 2, 2, 2, ... 무한 반복!
```

맨 앞에 값이 추가되어 무한 루프에 빠지는건 당연한 결과이다. 이런 실수를 왜 할까 싶을 수도 있지만 실무에서는 코드가 복잡해 지면 순회 중인 리스트인지 아닌지 확인이 어려울 수도 있다.

## 해결 방안

### 해결책 1: 복사본을 순회하고 원본을 수정

가능하면 수정하지 않는게 좋다. 하지만 알고리즘 특성상 꼭 수정해야 한다면, 복사본을 순회하면서 원본을 고치자.

```python
# 딕셔너리: 키만 복사
my_dict = {"red": 1, "blue": 2, "green": 3}
for key in list(my_dict.keys()):   # 키 목록 복사본을 순회
    if key == "blue":
        my_dict["green"] = 4        # 원본 수정 → 안전

# 리스트: 통째로 복사
my_list = [1, 2, 3]
for number in list(my_list):       # 복사본 순회
    if number == 2:
        my_list.insert(0, 4)        # 원본에 삽입 → 무한 루프 없음
# 결과: [4, 1, 2, 3]
```

### 해결책 2 (성능): 수정을 별도 컨테이너에 모았다가 나중에 병합

복사가 너무 느릴 만큼 컨테이너가 거대하다면, 변경 사항을 별도 컨테이너(스테이징)에 쌓아뒀다가 순회가 끝난 뒤 한 번에 병합하자.

```python
my_dict = {"red": 1, "blue": 2, "green": 3}
modifications = {}

for key in my_dict:
    if key == "blue":
        modifications["green"] = 4   # 별도 공간에 변경 사항 적재

my_dict.update(modifications)        # 순회 끝난 뒤 병합
```

## 한줄 요약

순회 중에는 컨테이너를 수정하지 마라
