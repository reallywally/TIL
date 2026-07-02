# Item 30 함수 인자가 변경될 수 있음을 알라

## 파이썬은 "참조에 의한 전달"

파이썬은 (C와의 연동을 빼면) 포인터 타입이 없다. 하지만 함수에 넘기는 인자는 모두 참조(reference)로 전달된다. 정수, 문자열 같은 단순(불변) 타입은 값으로 전달되는 것처럼 보이지만, 그건 불변 객체라 바꿀 수 없을 뿐이다. 더 복잡한 객체는 호출자의 의도와 무관하게, 넘겨받은 함수가 수정할 수 있다.

```python
def my_func(items):
    items.append(4)

x = [1, 2, 3]
my_func(x)
print(x)   # [1, 2, 3, 4]  ← 원본이 바뀜!
```

여기서 함수는 x라는 변수 자체를 다른 것으로 교체할 수는 없다(C 포인터처럼). 하지만 x가 가리키는 객체의 내용은 수정할 수 있다.

## 별칭(alias) 주의

변수에 변수를 대입하면, 같은 객체를 가리키는 **별칭(alias)**이 생긴다. 그래서 "별개처럼 보이는 변수"를 넘겨도 실제로는 원본이 바뀔 수 있다.

```python
a = [7, 6, 5]
b = a              # 별칭 생성 (같은 리스트를 가리킴)
my_func(b)
print(a)           # [7, 6, 5, 4]  ← a도 같이 바뀜
```

## 해결책: 방어적 복사

리스트, 딕셔너리는 복사본을 넘겨 함수의 동작으로부터 원본을 보호할 수 있다.
리스트는 시작·끝 인덱스 없는 슬라이스 [:]로 복사가 된다.

```python
def capitalize_items(items):
    for i in range(len(items)):
        items[i] = items[i].capitalize()

my_items = ["hello", "world"]
items_copy = my_items[:]        # 복사본 생성
capitalize_items(items_copy)
print(items_copy)               # ['Hello', 'World']
print(my_items)                 # ['hello', 'world']  ← 원본 그대로
```

딕셔너리는 전용 copy() 메서드로 복사할 수 있다.

```python
def concat_pairs(items):
    for key in items:
        items[key] = f"{key}={items[key]}"

my_pairs = {"foo": 1, "bar": 2}
pairs_copy = my_pairs.copy()    # 복사본 생성
concat_pairs(pairs_copy)
print(pairs_copy)               # {'foo': 'foo=1', 'bar': 'bar=2'}
```

### 참고

[:]와 .copy()는 **얕은 복사(shallow copy)**입니다. 중첩된 내부 객체까지 보호하려면 copy.deepcopy가 필요하다.

## 한줄 요약

파이썬 인자는 참조로 전달되고 받는 함수, 메서드가 그 속성을 수정할 수 있으니 함수가 입력 인자를 수정한다면 이름과 문서로 명확히 밝히고, 그 외에는 인자를 수정하지 말자.
