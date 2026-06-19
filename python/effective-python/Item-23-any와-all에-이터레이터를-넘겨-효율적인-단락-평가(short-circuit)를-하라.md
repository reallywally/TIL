# Item 23 any와 all에 이터레이터를 넘겨 효율적인 단락 평가(short-circuit)를 하라

## 상황 설정

쇼핑몰을 개발하고 있다고 가정해보자. 급하기 개발했지만 잘 운영되고 있었다. 그런데 최근 재고가 없는 상품이 결제되어 컴플레인이 들어왔고 이를 개선하고자 한다.  
개선 사항은 고객이 장바구니에 담은 상품들을 결제 전에 주문 가능한 상태인지 검증하는것이다. 그런 데 상품 하나의 재고를 확인하는 API를 호출하려 하니 시간이 너무 오래 걸렸다.

```python
import time

def check_stock(sku):
    """외부 재고 API 호출 — 느리고 비싼 연산이라고 가정"""
    time.sleep(0.3)              # 네트워크 지연 흉내
    print(f"  [API 호출] {sku} 재고 확인 중...")
    inventory = {"A-101": 5, "B-202": 0, "C-303": 12, "D-404": 3}
    return inventory.get(sku, 0)

def is_in_stock(sku):
    return check_stock(sku) > 0
```

## 나쁜 방법: 리스트 컴프리헨션으로 전부 먼저 계산

머릿속에 바로 생각나는 방법은 모든 모품의 재고 확인 API로 호출하고 결과를 리스트로 만든 다음 주물 불가능한 상품이 있는지 체크하는것이다.

```python
cart = ["A-101", "B-202", "C-303", "D-404"]

results = [is_in_stock(sku) for sku in cart]   # 4개 전부 API 호출
all_available = False not in results
```

문제는 두 번째 상품(B-202)이 품절이라 이미 답이 False로 정해졌는데도, C-303, D-404까지 쓸데없이 API를 다 호출한다는 것이다.

## 좋은 방법: all + 제너레이터 표현식

all은 falsey를 만나는 즉시 멈춘다. 제너레이터 표현식을 넘기면 품절 상품을 만난 순간 API 호출을 중단한다.

```python
cart = ["A-101", "B-202", "C-303", "D-404"]

all_available = all(is_in_stock(sku) for sku in cart)
# 출력:
#   [API 호출] A-101 재고 확인 중...   (재고 있음 → 계속)
#   [API 호출] B-202 재고 확인 중...   (품절! → 여기서 멈춤)
# C-303, D-404는 호출조차 안 함
```

## 제너레이터 함수로도 같은 효과

조건이 복잡하면 yield 제너레이터 함수로 빼도 좋다.

```python
def each_item_in_stock(cart):
    for sku in cart:
        yield is_in_stock(sku)

all_available = all(each_item_in_stock(cart))
```

## 반대 로직: any — "위험 상품이 하나라도 있나?"

이번엔 대부분 False이고 가끔만 True인 검사를 생각해보자. 예를 들어 장바구니에 배송 제한/판매 금지 상품이 하나라도 섞여 있는지 확인하는 경우이다. 이 경우 하나만 걸려도 결제를 막아야 하니, 첫 번째 위험 상품을 발견하는 즉시 멈춰야 한다.

```python
RESTRICTED = {"D-404"}   # 판매 제한 상품 목록

def is_restricted(sku):
    print(f"  [검사] {sku} 제한 여부 확인...")
    return sku in RESTRICTED

has_restricted = any(is_restricted(sku) for sku in cart)
```

혹은 "제한 상품이 없어야 결제 가능"은 이렇게 표현할 수 있다.

```python
can_checkout = not any(is_restricted(sku) for sku in cart)
```

## 한줄 요약: any vs all 어느 걸 쓸까

True로 일찍 끝내고 싶다면 any, False로 일찍 끝내고 싶면 → all을 사용하자.
