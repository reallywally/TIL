# Item 31: 세 개 초과를 언패킹하게 하지 말고 전용 결과 객체를 반환하라

## 시작: 함수는 여러 값을 반환하는 것처럼 보인다

언패킹 문법 덕분에 파이썬 함수는 여러 값을 반환하는 것처럼 쓸 수 있다. 숫자 리스트에서 최소, 최대를 구하는 코드를 예시로 보자.

```python
def get_stats(numbers):
    minimum = min(numbers)
    maximum = max(numbers)
    return minimum, maximum

lengths = [63, 73, 72, 60, 67, 66, 71, 61, 72, 70]
minimum, maximum = get_stats(lengths)   # 두 값 반환처럼 보임
```

실제로는 튜플로 함께 반환되고, 호출부가 언패킹으로 두 변수에 나눠 담는 것이다. 별표 표현식(Item 16)으로 중간을 몰아 받을 수도 있습니다.

```python
def get_avg_ratio(numbers):
    average = sum(numbers) / len(numbers)
    scaled = [x / average for x in numbers]
    scaled.sort(reverse=True)
    return scaled

longest, *middle, shortest = get_avg_ratio(lengths)
```

## 문제: 반환 값이 많아지면 위험해진다

요구가 늘어 평균, 중앙값, 개수까지 반환하게 됐다고 하면 코드가 아래와 같이 수정된다.

```python
def get_stats_more(numbers):
    minimum = min(numbers)
    maximum = max(numbers)
    count = len(numbers)
    average = sum(numbers) / count
    median = get_median(numbers)

    return minimum, maximum, average, median, count

minimum, maximum, average, median, count = get_stats_more(lengths)
```

두 가지 문제가 생깁니다.

### 첫째

반환 값이 전부 숫자라 순서를 바꿔도 티가 안 난다. 예를 들어 average와 median의 자리를 실수로 바꿔도 에러 없이 잘못된 값이 들어가, 나중에 찾기 힘든 버그가 된다.

```python
# 앗! median과 average가 뒤바뀜 — 에러 없이 조용히 틀림
minimum, maximum, median, average, count = get_stats_more(lengths)
```

### 둘째

호출, 언패킹 줄이 너무 길어져 PEP 8 때문에 여러 방식으로 줄바꿈해야 하고, 가독성이 떨어진다.

## 원칙: 언패킹은 세 개까지만

함수의 다중 반환 값을 언패킹할 때 **변수 4개 이상은 쓰지 마라**. 튜플, 또는 변수 2개 + 별표 1개, 또는 그보다 짧은 것까지만 사용하자.

## 그 이상이 필요하면

경량 클래스를 정의해 그 인스턴스를 반환하는 게 낫다. dataclasses가 딱이다.

```python
from dataclasses import dataclass

@dataclass
class Stats:
    minimum: float
    maximum: float
    average: float
    median: float
    count: int

def get_stats_obj(numbers):
    count = len(numbers)
    return Stats(
        minimum=min(numbers),
        maximum=max(numbers),
        count=count,
        average=sum(numbers) / count,
        median=get_median(numbers),
    )

result = get_stats_obj(lengths)
print(result)
# Stats(minimum=60, maximum=73, average=67.5, median=68.5, count=10)
```

각 값을 이름(필드명)으로 접근하니 순서 뒤바뀜 버그가 사라지고, 코드가 명확하며 나중에 리팩터링하기도 쉽다. (예: result.average, result.median)

## 한줄 요약

함수는 변수 4개 이상으로 언패킹하지 말고 그럴땐 경량 클래스의 인스턴스를 반환하자.
