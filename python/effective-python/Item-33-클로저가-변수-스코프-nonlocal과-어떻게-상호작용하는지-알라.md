# Item 33 클로저가 변수 스코프, nonlocal과 어떻게 상호작용하는지 알라

## 시작: 우선순위 정렬

숫자 리스트를 정렬하되 특정 그룹을 앞으로 보내고 싶다고 가정하자.(UI에서 중요 메시지를 먼저 보여줄 때 유용). sort의 key 인자로 헬퍼 함수를 넘기는 게 흔한 방법이다.

```python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        return (1, x)
    values.sort(key=helper)

numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
sort_priority(numbers, group)
print(numbers)   # [2, 3, 5, 7, 1, 4, 6, 8]
```

위 코드가 잘 동작하는 이유는 세 가지이다.

- 클로저(closure): 파이썬 함수는 자신이 정의된 스코프의 변수를 참조할 수 있다. 그래서 helper가 바깥의 group에 접근 가능하다.
- 함수는 일급 객체(first-class): 함수를 변수처럼 다루고 인자로 넘길 수 있어, sort의 key에 클로저를 넘길 수 있다.
- 시퀀스(튜플) 비교 규칙: 튜플은 인덱스 0부터 비교하고, 같으면 다음 인덱스를 비교한다. 그래서 (0, x) / (1, x) 반환이 두 그룹으로 나뉜 정렬을 만든다.

## 함정: 클로저 안에서 플래그를 바꾸려 하면

"우선순위 항목을 발견했는지"를 반환하고 싶어, 클로저 안에서 플래그를 켜보자.

```python
def sort_priority2(numbers, group):
    found = False
    def helper(x):
        if x in group:
            found = True        # 플래그를 켜려는 시도
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found

found = sort_priority2(numbers, group)
print("Found:", found)   # False  ← group 항목이 분명 있는데도 False!
```

정렬은 맞게 됐으니 group 항목이 있었던 게 분명한데, found는 False로 나온다. 왜일까?
