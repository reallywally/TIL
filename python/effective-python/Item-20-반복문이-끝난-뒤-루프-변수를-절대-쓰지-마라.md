# Item 20 반복문이 끝난 뒤 루프 변수를 절대 쓰지 마라

## 어떤 동작인가

파이썬에서는 for 반복문에 쓰는 순회 변수가 반복이 끝난 뒤에도 계속 남아 있다.

```python
for i in range(3):
    print(f"Inside {i=}")
print(f"After {i=}")
# Inside i=0 / Inside i=1 / Inside i=2 / After i=2
```

## 이 동작을 이용한 (위험한) 패턴

이 "끝난 뒤에도 변수가 남는" 성질을 일부러 활용하는 코드가 있다. 대표적으로 목록에서 원하는 항목의 인덱스를 찾고 싶을때 많이 사용한다.

```python
categories = ["Hydrogen", "Uranium", "Iron", "Other"]
for i, name in enumerate(categories):
    if name == "Iron":
        break
print(i)   # 2
```

만약 찾는 원소가 없으면 끝까지 돌고, 마지막 인덱스가 남아 "Other"(여기선 인덱스 3)로 분류된다.

```python
for i, name in enumerate(categories):
    if name == "Lithium":
        break
print(i)   # 3 (못 찾아서 끝까지 감)
```

## 무엇이 문제인가

위 코드는 "break로 일찍 끝나거나, 아니면 끝까지 돈다"는 두 경우만 가정했다.
그런데 사실 세 번째 경우가 있다.
바로 리스트가 처음부터 비어 있어서 반복이 한 번도 시작되지 않는 경우이다.
이러면 변수 자체가 만들어지지 않아 런타임 예외가 난다.

```python
categories = []
for i, name in enumerate(categories):
    if name == "Lithium":
        break
print(i)   # NameError: name 'i' is not defined
```

즉, 반복문 뒤에서 루프 변수에 접근할 때 그 변수가 존재한다고 **항상 보장할 수 없다**.
그래서 실무에선 이런 식으로 쓰지 않는 게 최선이다.

## 다행?인 점: 컴프리헨션은 변수가 새지 않는다

다행히(혹은 불행히) 리스트 컴프리헨션과 제너레이터 표현식은 이 변수 누수(leak)가 없다.
그래서 앞의 코드와 다르게 내부 변수에 밖에서 접근하면 항상 없으므로, 이 함정에 빠질 일이 없다.

```python
my_numbers = [37, 13, 128, 21]
found = [i for i in my_numbers if i % 2 == 0]
print(i)   # NameError: name 'i' is not defined (항상)
```

## 한줄 요약

반복문 안에서 사용한 변수는 끝나고 재사용 하지말자.
