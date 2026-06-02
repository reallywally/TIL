# item 15 한 슬라이스 안에서 stride와 인덱스를 같이 쓰지 마라

## 핵심 개념

python에는 슬라이싱 기본형(somelist[start:end]) 외에, **stride(보폭)**를 지정하는 somelist[start:end:stride] 문법이 있다. 이걸로 n개마다 하나씩 골라낼 수 있다.

```python
x = ["red", "orange", "yellow", "green", "blue", "purple"]
odds = x[::2]  # 0,2,4번째 → ['red', 'yellow', 'blue']
evens = x[1::2]  # 1,3,5번째 → ['orange', 'green', 'purple']

print("Odds: ", odds)
print("Evens:", evens)
```

그리고 stride를 많이 활용되는 트릭으로 -1을 주면 시퀀스를 뒤집힌다.

```python
x = ["red", "orange", "yellow", "green", "blue", "purple"]
reversed_x = x[::-1]  # 역순 → ['purple', 'blue', 'green', 'yellow', 'orange', 

print("Reversed:", reversed_x)
```

## 무엇이 문제인가

설명만 보면 특별한 문제는 없어 보인다. 하지만 start, end, stride 세 개를 한 슬라이스 안에 다 넣으면 읽기고 생각하기가 매우 어려워진다. 특히 stride가 음수일 때 start/end가 어떤 식으로 적용되는지 직관적이지 않다.

```python
x = ["a", "b", "c", "d", "e", "f", "g", "h"]
x[2::2]      # ['c', 'e', 'g']
x[-2::-2]    # ['g', 'e', 'c', 'a']
x[-2:2:-2]   # ['g', 'e']
x[2:2:-2]    # []   ← 헷갈리쥬?
```

밀도가 높아 가독성이 떨어지고 버그의 원인이 되어 start, end, stride 세개를 한번에 사용하는건 권장되지 않는다.

## 권장 사항

책에서 말하는 권장사항은 다음과 같다.

- stride와 start/end 인덱스를 한 표현식에 같이 쓰지 말자.
- stride를 꼭 써야 한다면 양수로 쓰고 start/end는 생략해라.
- 셋 다 필요하면 두 줄로 나눠라 — 한 번은 striding, 한 번은 slicing.

```python
y = x[::2]    # 먼저 stride만:  ['a', 'c', 'e', 'g']
z = y[1:-1]   # 그다음 slicing: ['c', 'e']
```

단, 이렇게 두 단계로 하면 데이터의 얕은 복사본(shallow copy)이 하나 더 생긴다. 그래서 첫 연산에서 최대한 결과 크기를 줄여두는 게 좋다.

## 한줄 요약

stride와 start/end 인덱스를 한 표현식에 같이 쓰지 말고 나눠 쓰자
