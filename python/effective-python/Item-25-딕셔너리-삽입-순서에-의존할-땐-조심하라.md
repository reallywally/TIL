# Item 25 딕셔너리 삽입 순서에 의존할 땐 조심하라

## 과거(3.5 이전): 딕셔너리는 순서가 뒤죽박죽

파이썬 3.5 이전에는 딕셔너리를 순회하면 삽입 순서와 무관하게 임의의 순서로 키가 나왔다. 게다가 프로그램을 다시 실행할 때마다 순서가 무작위로 바뀌기도 했다.

```python
# Python 3.5
baby_names = {"cat": "kitten", "dog": "puppy"}
print(baby_names)
# {'dog': 'puppy', 'cat': 'kitten'}   ← 넣은 순서와 반대!
```

이는 당시 해시 테이블 구현이 hash 함수와 프로세스 시작 시 정해지는 랜덤 시드를 함께 썼기 때문입니다. 그래서 테스트 재현이나 디버깅이 어려웠고, 입문자에게 특히 혼란스러웠다.

## 현재(3.7+): 삽입 순서가 보장된다

파이썬 3.6부터 동작이 바뀌었고, 3.7부터는 공식 언어 명세가 되었다. 이제 딕셔너리는 넣은 순서를 그대로 유지한다.

```python
baby_names = {"cat": "kitten", "dog": "puppy"}
print(baby_names)
# {'cat': 'kitten', 'dog': 'puppy'}   ← 넣은 순서 그대로
```

keys(), values(), items(), popitem()도 모두 삽입 순서를 따른다. 이 변화는 딕셔너리에 의존하는 다른 기능에도 적용됩니다.

- **kwargs: 키워드 인자가 호출 순서대로 들어옴
- 인스턴스 속성(__dict__): 클래스 인스턴스의 필드도 대입한 순서대로 유지됨

이제 이 순서 보장을 믿고 코드를 짤 수 있고, 심지어 내가 설계하는 API의 동작으로 삼아도 됩니다.

### 참고

collections.OrderedDict는 예전부터 순서를 유지했다. 일반 dict와 동작은 비슷하지만 성능 특성이 다르다. 키 삽입과 popitem이 매우 빈번한 경우(예: LRU 캐시 구현)엔 OrderedDict가 더 나을 수 있다.

## 함정: "딕셔너리처럼 보이는 것"은 순서를 보장하지 않는다

문제는, 파이썬에서는 표준 dict가 아니면서 딕셔너리처럼 행동하는 커스텀 타입을 쉽게 만들 수 있다는 점입이다. 파이썬은 덕 타이핑(duck typing)을 쓰니, 이런 객체를 dict 자리에 넣어도 에러가 안 나지만 동작이 달라질 수 있디. 귀여운 동물 투표 순위를 매기는 예를 봅시다.

```python
votes = {"otter": 1281, "polar bear": 587, "fox": 863}

def populate_ranks(votes, ranks):
    names = list(votes.keys())
    names.sort(key=votes.get, reverse=True)
    for i, name in enumerate(names, 1):
        ranks[name] = i          # 1위부터 순서대로 넣음

def get_winner(ranks):
    return next(iter(ranks))     # "첫 번째 키가 1위"라고 가정

#ㅍ표준 dict로는 잘 동작합니다.
pythonranks = {}
populate_ranks(votes, ranks)
print(ranks)              # {'otter': 1, 'fox': 2, 'polar bear': 3}
print(get_winner(ranks))  # otter
```

그런데 요구사항이 "UI에 알파벳 순으로 보여달라"로 바뀌어, 순회 시 키를 정렬해 내놓는 딕셔너리 유사 클래스를 만들었다고 가정하다.

```python
from collections.abc import MutableMapping

class SortedDict(MutableMapping):
    def __init__(self):
        self.data = {}
    def __getitem__(self, key):
        return self.data[key]
    def __setitem__(self, key, value):
        self.data[key] = value
    def __delitem__(self, key):
        del self.data[key]
    def __iter__(self):
        for key in sorted(self.data.keys()):   # 정렬해서 순회
            yield key
    def __len__(self):
        return len(self.data)
```

이걸 dict 대신 넣으면 에러는 없지만 결과가 틀리다.

```python
sorted_ranks = SortedDict()
populate_ranks(votes, sorted_ranks)
print(get_winner(sorted_ranks))   # fox   ← 알파벳상 첫 번째라서!
```

```get_winner```가 "순회 첫 키 = 1위"라는 삽입 순서 가정을 했는데, SortedDict는 알파벳 순으로 순회하니 가정이 깨진 것이다.

## 해결책 세 가지

### 1. 순서에 의존하지 않게 코드를 다시 쓰기

가장 보수적이지만 가장 견고한 방식이다.

```python
def get_winner(ranks):
    for name, rank in ranks.items():
        if rank == 1:
            return name
# → otter (정확)
```

### 2. 런타임에 타입을 명시적으로 검사

성능적으로는 더 나음

```python
def get_winner(ranks):
    if not isinstance(ranks, dict):
        raise TypeError("must provide a dict instance")
    return next(iter(ranks))
# → SortedDict를 주면 TypeError
```

### 3. 타입 애너테이션 + 정적 분석(mypy)으로 강제

```get_winner(ranks: dict[str, int])```로 표기하면, SortedDict를 넘기는 코드를 mypy --strict가 컴파일 전에 에러로 잡아준다. 정적 타입 안전성과 런타임 성능을 모두 챙기는 방법이다.

## 한줄 요약

제곧내(딕셔너리 삽입 순서에 의존할 땐 조심하라).
