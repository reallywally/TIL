# Item 21 인자를 순회할 때는 방어적으로 코딩하라

## 문제 상황: 이터레이터는 한 번만 순회된다

함수가 리스트를 받아 여러 번 순회해야 할 때가 많다. 예를 들어 도시별 관광객 수에서 각 도시의 비율을 구하는 함수를 만들어 보자.

```python
def normalize(numbers):
    total = sum(numbers)          # 1차 순회
    result = []
    for value in numbers:         # 2차 순회
        result.append(100 * value / total)
    return result

visits = [15, 35, 80]
print(normalize(visits))   # [11.54..., 26.92..., 61.54...]
```

방문객 목록을 list로 넘기면 아무런 문제가 발생하지 않는다.
하지만 만약 방문객 데이터가 많아서 제너레이터로 파일을 읽으면 문제가 발생한다.

```python
def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)

it = read_visits("my_numbers.txt")  # it은 제너레이터 객체
print(normalize(it))   # []  ← 빈 결과!
```

자바로 만들었다면 상상도 못할 상황이다. 아무튼 빈 결과가 나오는 원인을 자세히 설명하면 it가 제너레이터 객체라서 ```normalize```함수의 sum()으로 1차 순회를 돌때 이미 파라미터로 받은 numbers가 빈 list로 된다. 그래서 2차 순회는 빈 list라서 아에 돌지를 않는다.
단어로 정확하게 설명하면 코드가 "원래 비어 있는 것"과 "소진된 것"을 구분하지 못해서 발생한 상황이다.

## 해결 방법 1: 입력을 리스트로 복사 (방어적 복사)

파라미터로 받은 list를 복사하여 1차 순회와 2차 순회를 분리하는것이다.
동작은 하지만, 복사본이 메모리를 엄청 차지할 수 있습니다. 그리고 반복이 필요할 때 마다 코드가 수정되어야할 수 있으며 애초에 제너레이터로 만든 의미가 사라진다.

```python
def normalize_copy(numbers):
    numbers_copy = list(numbers)   # 통째로 복사
    total = sum(numbers_copy)
    return [100 * v / total for v in numbers_copy]
```

## 방법 2: 새 이터레이터를 만드는 함수를 받기

호출할 때마다 새 제너레이터를 주는 람다를 넘기는 방법이다. 역시 동작은 하지만 람다를 매번 넘기는 게 번거롭고 코드만 보면 왜 람담다로 넘기는지 이해가 되지 않는다.

```python
def normalize_func(get_iter):
    total = sum(get_iter())                 # 새 이터레이터
    return [100 * v / total for v in get_iter()]  # 또 새 이터레이터

path = "my_numbers.txt"
percentages = normalize_func(lambda: read_visits(path))
```

## 방법 3 (권장): 이터레이터 프로토콜을 구현한 컨테이너 클래스

더 나은 방법은 이터러블 컨테이너 클래스를 정의하는 것이다.
```for x in foo```를 만나면 파이썬은 ```iter(foo) → foo.__iter__()```를 호출해 이터레이터를 얻고, next()를 StopIteration이 날 때까지 반복한다.
```__iter__```를 제너레이터로 구현하면 이 모든 동작을 직접 누릴 수 있다.

```python
class ReadVisits:
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

visits = ReadVisits(path)
print(normalize(visits))   # 원본 normalize 그대로 동작!
```

normalize의 sum이 ```__iter__```를 호출해 새 이터레이터를 만들고, for도 또 ```__iter__```를 호출해 또 다른 새 이터레이터를 만든다.
각각 독립적으로 끝까지 돌기 때문에 매 순회가 전체 데이터를 본다.
유일한 단점은 입력 데이터를 여러 번 읽는다는 것뿐.

## 이터레이터를 거부하는 방어적 코딩

이제 방법3를 이용하여 "이터레이터가 아니라 컨테이너만 받겠다"고 강제할 수 있다.
이렇게 방어 코딩을 하여 앞서 설명했던 문제 상황을 방지할 수 있다.

### 방어 코딩 방법1

```python
def normalize_defensive(numbers):
    if iter(numbers) is numbers:   # 이터레이터면 자기 자신을 반환
        raise TypeError("Must supply a container")
    total = sum(numbers)
    return [100 * v / total for v in numbers]
```

### 방어 코딩 방법2

```python
from collections.abc import Iterator

def normalize_defensive(numbers):
    if isinstance(numbers, Iterator):
        raise TypeError("Must supply a container")
```

## 한줄 요약

반복문으로 순회할때는 방어적으로 코딩하자
