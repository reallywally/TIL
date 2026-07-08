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

## 원인: 변수 참조 vs. 대입의 스코프 규칙

**변수를 참조(읽기)**할 때 인터프리터는 함수 → 감싸는 바깥 함수들 → 모듈(전역) → 내장 스코프(len, str 등) 순으로 스코프를 탐색하고 어디에도 없으면 NameError가 발생한다.
그런데 **변수에 대입(쓰기)**은 다르게 동작합니다. 현재 스코프에 이미 있으면 그 값을 바꾸지만, 없으면 그 대입을 "새 변수 정의"로 취급한다. 중요한 건, 새로 정의된 변수의 스코프가 바깥이 아니라 대입이 일어난 **그 함수라는 점**이다.
즉, helper 안의 found = True는 sort_priority2의 found를 바꾸는 게 아니라, helper 안에 새로운 **지역 변수 found를 만든 것**이다. 그래서 바깥 found는 그대로 False이다.
이걸 흔히 스코핑 버그라고 부르는데, 사실 이건 의도된 동작이다. 함수 안의 대입이 바깥(모듈 전역)을 오염시키지 못하게 막아, 전역이 쓰레기 값으로 넘치는 걸 방지할 수 있다.

## 해결책 1: nonlocal

클로저 스코프 밖으로 대입하려면 ```nonlocal``` 문을 사용한다. 해당 변수 이름에 대해 대입 시 스코프를 위로 탐색하라는 표시이다. 단 전역 오염 방지를 위해 모듈 전역까지는 올라가지 않는다.

```python
def sort_priority3(numbers, group):
    found = False
    def helper(x):
        nonlocal found          # 추가
        if x in group:
            found = True
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found

found = sort_priority3(numbers, group)
print("Found:", found)   # True
```

```nonlocal```은 "데이터가 클로저 밖 다른 스코프로 대입된다"는 걸 드러낸다. 변수 대입을 모듈 스코프로 직접 보내는 global 문과 상호 보완적이다.

### ⚠️주의

다만 전역 변수 안티패턴과 마찬가지로, ```nonlocal```도 간단한 함수 이상에는 쓰지 않길 권증된다. 부작용을 따라가기 어렵고, 특히 긴 함수에서 ```nonlocal``` 선언과 실제 대입이 멀리 떨어져 있으면 이해하기 힘들다

## 해결책 2 (복잡해지면): 상태를 헬퍼 클래스로 감싸기

```nonlocal```이 복잡해지기 시작하면, 상태를 클래스로 감싸는 게 낫다. __call__을 구현해 함수처럼 호출 가능한 객체로 만들면 같은 결과를 얻을 수 있다.

```python
class Sorter:
    def __init__(self, group):
        self.group = group
        self.found = False

    def __call__(self, x):
        if x in self.group:
            self.found = True
            return (0, x)
        return (1, x)

sorter = Sorter(group)
numbers.sort(key=sorter)
print("Found:", sorter.found)   # True
```

코드는 조금 길지만 훨씬 이해하기 쉽고 확장하기 좋다. 결과는 sorter.found 속성으로 접근하면 된다.

## 한줄요약

클로저 함수는 자신이 정의된 바깥 스코프들의 변수를 참조할 수 있지만 \대입으로 바깥 스코프에 영향을 줄 수 없다.

## 여담

VSCode에서 type check mode를 strict으로 하고 익스텐션으로 ruff + error lens 쓰면 화면에서 관련 내용으로 경고가 표시된다.
