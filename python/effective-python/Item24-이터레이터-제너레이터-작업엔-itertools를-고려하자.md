# Item 24 이터레이터 제너레이터 작업엔 itertools를 고려하라

파이썬의 itertools 내장 모듈은 이터레이터를 다루는 유용한 함수가 아주 많다.
순회 코드를 짜면서 있으면 좋을 법한 기능이 있으면 공식 문서를 다시 들춰볼 가치가 있다.
함수들은 크게 세 가지 범주로 나눌 수 있고 하나씩 살펴보자.

```python
import itertools
```

## 1. 이터레이터 연결하기

### chain

여러 이터레이터를 하나로 이어 붙인다.(평탄화)

```python
it = itertools.chain([1, 2, 3], [4, 5, 6])
print(list(it))   # [1, 2, 3, 4, 5, 6]
```

### chain.from_iterable

"이터레이터들의 이터레이터"를 받아 하나로 평탄화한다.

```python
nested = [["a", "b"], ["c", "d"]]
print(list(itertools.chain.from_iterable(nested)))   # ['a', 'b', 'c', 'd']
```

### repeat

한 값을 무한히(또는 두 번째 인자로 횟수 지정) 반복한다.

```python
print(list(itertools.repeat("hello", 3)))   # ['hello', 'hello', 'hello']
```

### cycle

이터레이터의 항목들을 무한히 순환한다.

```python
it = itertools.cycle([1, 2])
print([next(it) for _ in range(6)])   # [1, 2, 1, 2, 1, 2]
```

### tee

하나의 이터레이터를 N개의 병렬 이터레이터로 복제한다.(각 이터레이터의 진행 속도가 다르면 버퍼링 때문에 메모리 사용이 늘 수 있음)

```python
it1, it2, it3 = itertools.tee(["a", "b"], 3)
print(list(it1), list(it2), list(it3))   # ['a','b'] ['a','b'] ['a','b']
```

### zip_longest

zip의 변형으로 길이가 다르면 짧은 쪽을 fillvalue로 채운다.(Item 18 참고)

```python
keys = ["one", "two", "three"]
values = [1, 2]
print(list(itertools.zip_longest(keys, values, fillvalue=0)))
# [('one', 1), ('two', 2), ('three', 0)]
```

## 2. 항목 걸러내기

### islice

이터레이터를 복사 없이 숫자 인덱스로 슬라이싱한다. 문법은 (end), (start, end), (start, end, step) 4가지 형태로 가능하다.

```python
values = list(range(1, 11))   # 1..10
print(list(itertools.islice(values, 5)))        # [1, 2, 3, 4, 5]
print(list(itertools.islice(values, 2, 8, 2)))  # [3, 5, 7]
```

### takewhile

술어(predicate)가 False가 될 때까지의 항목만 반환.

```python
values = list(range(1, 11))
print(list(itertools.takewhile(lambda x: x < 7, values)))   # [1,2,3,4,5,6]
```

예시만 보면 filter와 혼동될 수도 있지만 엄연히 다르다. takewhile은 설명대로 False가 되면 멈추는데 filter는 전체에서 걸러준다.

```python
values = [1, 2, 10, 3, 4, 5, 6]  # 예시 목록 중간에 10이 있음
print(list(itertools.takewhile(lambda x: x < 7, values)))  # [1,2]
print(list(filter(lambda v: v < 7, values))) # [1, 2, 3, 4, 5, 6]
```

### dropwhile

takewhile의 반대. 술어가 처음 False가 되는 지점부터 끝까지 반환.

```python
print(list(itertools.dropwhile(lambda x: x < 7, values)))   # [7, 8, 9, 10]
```

### filterfalse

내장 filter의 반대. 술어가 False인 항목만 반환.

```python
evens = lambda x: x % 2 == 0
print(list(filter(evens, values)))                  # [2, 4, 6, 8, 10]
print(list(itertools.filterfalse(evens, values)))   # [1, 3, 5, 7, 9]
```

## 3. 항목 조합 만들기

### batched

파이썬 3.12 부터 사용 가능한 기능으로 입력을 고정 크기의 겹치지 않는 묶는다. 데이터를 일괄 처리하거나 크기 제한을 맞출 때 유용하다. 묶는 사이즈로 마지막 묶음은 더 작을 수 있다.

```python
print(list(itertools.batched(range(1, 10), 3)))   # [(1,2,3),(4,5,6),(7,8,9)]
print(list(itertools.batched([1, 2, 3], 2)))      # [(1, 2), (3,)]
```

### pairwise

인접한 쌍을 순회(겹침 포함)한다. 그래프 경로 탐색 등에 유용하고 파이썬 3.10이후 부터 사용 가능하다.

```python
route = ["LA", "Bakersfield", "Modesto", "Sacramento"]
print(list(itertools.pairwise(route)))
# [('LA','Bakersfield'), ('Bakersfield','Modesto'), ('Modesto','Sacramento')]
```

### accumulate

두 인자 함수를 누적 적용하며 매 단계의 누적값을 출력한다. 함수를 안 주면 기본은 합계이다. (functools.reduce와 비슷하되 한 단계씩 내놓음.)

```python
values = list(range(1, 11))
print(list(itertools.accumulate(values)))   # [1, 3, 6, 10, 15, 21, ...]

def sum_mod_20(a, b):
    return (a + b) % 20
print(list(itertools.accumulate(values, sum_mod_20)))  # [1,3,6,10,15,1,8,16,5,15]
```

### product

하나 이상의 이터레이터의 데카르트 곱을 한다. 깊게 중첩된 컴프리헨션의 좋은 대안이다.

```python
print(list(itertools.product([1, 2], repeat=2)))   # [(1,1),(1,2),(2,1),(2,2)]
print(list(itertools.product([1, 2], ["a", "b"]))) # [(1,'a'),(1,'b'),(2,'a'),(2,'b')]
```

### permutations

길이 N의 순열(순서 구분).

```python
print(list(itertools.permutations([1, 2, 3], 2)))
# [(1,2),(1,3),(2,1),(2,3),(3,1),(3,2)]
```

### combinations

길이 N의 조합(순서 무시, 중복 없음).

```python
print(list(itertools.combinations([1, 2, 3, 4], 2)))
# [(1,2),(1,3),(1,4),(2,3),(2,4),(3,4)]
```

### combinations_with_replacement

같은 값의 중복을 허용하는 조합.

```python
print(list(itertools.combinations_with_replacement([1, 2, 3], 2)))
# [(1,1),(1,2),(1,3),(2,2),(2,3),(3,3)]
```

## 현실

일반적인 웹 개발, 백엔드 서버를 하면서 3번 조합하기를 활용할 일은 거의 없을거 같다.
1번 연결하기와 2번 걸러내기는 그래도 활용하기 좋지만 모든 기능을 활용하기도 쉽지 않다.
그러면 일반적으로 많이 사용할법한 기능을 무엇일까?
클로드에게 itertools로 리팩토링할 수 있는 부분이 있는지 검토시키는 것도 좋은 방법 같다.
지금은 쇼핑몰을 예시로 클로드가 말한 대표적인 것들은 살펴보자.

### 1. batched — 대량 처리

주문 알림, 재고 동기화, DB 일괄 삽입, 외부 API 호출은 한 건씩 보내면 느리고, 한 번에 다 보내면 한도 초과가 난다. 그래서 N개씩 묶어 보내는 게 좋다.

```python
order_ids = [1001, 1002, 1003, 1004, 1005, 1006, 1007]

# 외부 배송사 API가 한 번에 최대 3건까지만 받는다고 가정
def send_to_courier(batch):
    print(f"배송 요청 전송: {batch}")

for batch in itertools.batched(order_ids, 3):
    send_to_courier(batch)

# 배송 요청 전송: (1001, 1002, 1003)
# 배송 요청 전송: (1004, 1005, 1006)
# 배송 요청 전송: (1007,)
```

### 2. product — 상품 옵션(SKU) 조합 생성

옵션이 있는 상품은 "사이즈 × 색상 × 소재"처럼 모든 조합의 SKU를 만들어야 한다고 가정하자. 중첩 for문 대신 product가 깔끔합니다. 예시는 zip써도 좋을거 같다.

```python
sizes = ["S", "M", "L"]
colors = ["Black", "White"]

for size, color in itertools.product(sizes, colors):
    sku = f"TSHIRT-{size}-{color[:2].upper()}"
    print(sku)

# TSHIRT-S-BL, TSHIRT-S-WH, TSHIRT-M-BL, TSHIRT-M-WH, TSHIRT-L-BL, TSHIRT-L-WH
```

### 3. islice — 페이지네이션

특정 목록을 페이지 단위로 잘라 보여줄 때, 특히 제너레이터(DB 커서·스트림)를 리스트로 다 만들지 않고 필요한 구간만 떼어낼 수 있어 메모리에 좋다.

```python
def all_products():
    """DB에서 상품을 하나씩 흘려주는 제너레이터라고 가정"""
    for i in range(1, 101):
        yield f"product-{i}"

def get_page(source, page, size=10):
    start = (page - 1) * size
    return list(itertools.islice(source, start, start + size))

print(get_page(all_products(), page=2))   # product-11 ~ product-20
```

### 4. chain / chain.from_iterable — 여러 소스 합치기

리스트를 +로 새로 만들지 않아 메모리 효율적이다.

```python
seoul_warehouse = ["A-101", "A-102"]
busan_warehouse = ["B-201", "B-202", "B-203"]

for sku in itertools.chain(seoul_warehouse, busan_warehouse):
    print("정산 처리:", sku)

# 창고가 여러 개라 리스트의 리스트로 들어올 때
warehouses = [seoul_warehouse, busan_warehouse]
all_skus = list(itertools.chain.from_iterable(warehouses))
print(all_skus)   # ['A-101', 'A-102', 'B-201', 'B-202', 'B-203']
```

### 5. accumulate — 누적 매출/장바구니 합계

매출 대시보드의 누적 추이나, 장바구니 항목을 하나씩 더해가는 누적 금액 계산에 잘 맞는다.

```python
daily_sales = [120, 200, 150, 300, 250]   # 일별 매출(만원)

cumulative = list(itertools.accumulate(daily_sales))
print(cumulative)   # [120, 320, 470, 770, 1020]  ← 그래프 누적선에 바로 사용
```

## 한줄 요약

itertools를 잘 활용 하자
