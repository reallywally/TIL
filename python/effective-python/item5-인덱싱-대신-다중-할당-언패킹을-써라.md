# item 5: 인덱싱 대신 다중 할당 언패킹을 써라

## 언패킹으로 더 깔끔하게

python에는 tuple이라는 자료구조가 있다. 그런데 list를 사용하던 습관이 있어서 tuple의 장점을 살리지 못하고 인덱싱으로 접근하는 안좋은 방법을 사용할 때가 있다. 언팩킹을 사용하면 한 줄로 처리할 수 있고, 변수 이름이 의미를 바로 드러나게 사용할 수 있다.

```python
item = ("Peanut butter", "Jelly")

# 인덱스 방식 (나쁜 예)
first = item[0]
second = item[1]

# 언패킹 방식 (좋은 예)
first, second = item
print(first, "and", second)
# → Peanut butter and Jelly
```

## 중첩 언패킹

권장하는 코드는 아니지만 언팩킹 문법이 얼마나 유연한지 보여주는 예시도 참고 하자.

```python
favorite_snacks = {
    "salty":  ("pretzels", 100),
    "sweet":  ("cookies",  180),
    "veggie": ("carrots",   20),
}

# 중첩 tuple도 한 번에 언패킹
((type1, (name1, cals1)),
 (type2, (name2, cals2)),
 (type3, (name3, cals3))) = favorite_snacks.items()

print(f"Favorite {type1} is {name1} with {cals1} calories")
# → Favorite salty is pretzels with 100 calories
```

## 언패킹으로 값 교환 (swap)

언패킹의 가장 실용적인 예시이다. java로 만들었다면 temp같은 임시 변수가 필요하지만 python의 언팩킹을 이용하면 임시 변수 없이 바로 값을 교환할 수 있다. 책 예시 코드는 버블 정렬인데 지금은 간단히 값 교체를 어떻게 하는지만 보자.

```python
simple_list = [1, 2]
print("Before swap:", simple_list)
# Before swap: [1, 2]

simple_list[0], simple_list[1] = simple_list[1], simple_list[0]
print("After swap:", simple_list)
# After swap: [2, 1]
```

## for 루프에서의 언패킹

for문에서 enumerate랑 같이 활용하면 동일한 작업을 하더라도 훨씬 가독성 좋은 코드로 작성할 수 있다.

```python
snacks = [("bacon", 350), ("donut", 240), ("muffin", 190)]

# 가능은 하지만 지저분하게 데이터를 꺼내 사용
for i in range(len(snacks)):
    item     = snacks[i]
    name     = item[0]
    calories = item[1]
    print(f"#{i+1}: {name} has {calories} calories")

# 명확하게 꺼내서 사용
for rank, (name, calories) in enumerate(snacks, 1):
    print(f"#{rank}: {name} has {calories} calories")
```

## 만약 속성이 많으면?

예시를 보면서 tuple이 간단한 구조이다. 그렇다면 만약 10개 넘는 속성이 있다면 동일하게 언팩킹으로 꺼내쓰면 될까? 사실 이정도 수준이 되면 tuple이 아니라 dataclass, NamedTuple, BaseModel 등 class로 처리하는게 좋다. 명확히 속성 몇개부터 class로 처리하라는 가이드는 없지만 보통 5개만 넘어도 힘드니 무조건 언팩킹을 위해 tuple을 사용할 필요는 없다.

## 한줄 요약

Python에는 한 번에 여러 값을 변수에 할당하는 언패킹 문법이 있으며, 시각적 노이즈가 줄고 코드가 명확해진다.
