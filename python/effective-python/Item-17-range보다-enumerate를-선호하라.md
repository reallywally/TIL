# Item 17 range보다 enumerate를 선호하라

## range의 용도

range는 정수 집합을 순회하는 반복문에 유용하다. java의 for문과 같고 아래 예와 같이 사용할 수 있다.

```python
for i in range(10):
    print(i)
```

리스트 같은 자료구조를 순회할 때도 유용하다.

```python
flavor_list = ["vanilla", "chocolate", "pecan", "strawberry"]
for flavor in flavor_list:
    print(f"{flavor} is delicious")
```

## 문제: 인덱스도 같이 필요할 때

리스트를 순회하면서 현재 항목의 인덱스도 알고 싶을 때가 많다. 그리고 흔히 range(len(...))로 인덱스를 만들고 다시 인덱싱하는 방식을 사용한다.

```python
for i in range(len(flavor_list)):
    flavor = flavor_list[i]
    print(f"{i + 1}: {flavor}")
```

리스트 길이를 구해야 하고, 배열에 인덱싱도 해야하는게 동작은 하지만 이건 어딘가 어설프다.

## 해결책: enumerate

파이썬은 enumerate 내장 함수를 제공한다. 어떤 이터레이터든 게으른(lazy) 제너레이터로 감싸서, (인덱스, 값) 쌍을 차례로 내놓는다.

```python
it = enumerate(flavor_list)
print(next(it))   # (0, 'vanilla')
print(next(it))   # (1, 'chocolate')
```

이 쌍을 for 문에서 바로 언패킹하면 훨씬 깔끔해진다.

```python
for i, flavor in enumerate(flavor_list):
    print(f"{i + 1}: {flavor}")
```

## 한줄 요약

인덱스와 값을 같이 사용한 반복분을 사용할때는 enumerate를 이용하자
