# item8 대입 표현식으로 반복을 피하라

## 바다코끼리 연산자(walrus operator)

### 기본 개념

python 3.8부터 바다코끼리 연산자라고도 불리는 ```:=``` 구문을 사용할 수 있다. 작동방식은 표현식의 결과를 변수에 할당하고, 동시에 반환한다.

- 일반 할당: a = b (a equals b)
- 대입 표현식: a := b (a walrus b)

### 패턴1. if문에서의 중복 제거

```python
count = fresh_fruit.get("lemon", 0)  # if 위에 선언
if count:
    make_lemonade(count)
else:
    out_of_stock()
```

기존 방식에 많이 쓰는 방식을 보면 count가 if문 위에 있다. 하지만 count는 사실 if 블록 안에서만 쓰이는데, 밖에서 선언되어 있어서 코드 읽는 사람이 "이 변수가 아래에서도 쓰이나?" 하고 헷갈릴 수도 있다. 이럴때 바다코끼리 연산자를 쓰면 조금 더 명확하게 코드를 작성할 수 있다.

```python
# walrus 방식 — 의도가 명확:
if count := fresh_fruit.get("lemon", 0):
    make_lemonade(count)
else:
    out_of_stock()
```

### 패턴2. 비교 조건이 있을 때

단순 truthy 체크가 아니라 숫자 비교가 필요할 때는 괄호로 감싸면 역시 한줄에 해결 할 수 있다.

```python
# 사이다를 만들려면 사과가 4개 이상 필요
if (count := fresh_fruit.get("apple", 0)) >= 4:
    make_cider(count)
else:
    out_of_stock()
```

### 패턴3. 임시 변수를 사용할때

python 3.8 이전에는 아래와 같이 하나씩 임시 변수로 값을 꺼내서 사용했었다. 물론 python 3.10부터는 match가 있어서 쉽게 해결할 수 있지만 그전에는 이런 방식을 사용했고 비슷하게 임시 변수로 꺼내 쓰는 경우에 바다코끼리 연산자는 좋은 해결 방법이 될 수 있다.

```python
# 기존 방식
count = fresh_fruit.get("banana", 0)
if count >= 2:
    pieces = slice_bananas(count)
    to_enjoy = make_smoothies(pieces)
else:
    count = fresh_fruit.get("apple", 0)
    if count >= 4:
        to_enjoy = make_cider(count)
    else:
        count = fresh_fruit.get("lemon", 0)
        if count:
            to_enjoy = make_lemonade(count)
        else:
            to_enjoy = "Nothing"

# walrus 방식 — switch/case처럼 평탄하게:
if (count := fresh_fruit.get("banana", 0)) >= 2:
    pieces = slice_bananas(count)
    to_enjoy = make_smoothies(pieces)
elif (count := fresh_fruit.get("apple", 0)) >= 4:
    to_enjoy = make_cider(count)
elif count := fresh_fruit.get("lemon", 0):
    to_enjoy = make_lemonade(count)
else:
    to_enjoy = "Nothing"
```

### 패턴4. do/while 루프 대용

python에는 아직도 do/while문이 없다. 솔직히 불편한적은 없었는데 아래와 같이 해결할 수 있다는 정도만 알고 가자.

```python
while fresh_fruit := pick_fruit():
    for fruit, count in fresh_fruit.items():
        batch = make_juice(fruit, count)
        bottles.extend(batch)
```

## 한줄 요약

바다코끼리 연산자를 이용해서 중복을 피하고 당연히 남발하지 말자
