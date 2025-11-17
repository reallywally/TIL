# Python Mutable Default Argument의 함정과 해결법

오늘 주제는 클로드씨에게 부탁해서 TIL에 쓸만한 컨텐츠를 추천 받아 작성한다. 나는 아직 이런 경험이 없지만 코드만 보면 이해가 안되는 부분이 있어 겸사겸사 내용을 작성한다.

## 상황

아래 예시 코드를 보자

```python
def add_item(item, item_list=[]):
    item_list.append(item)
    return item_list


a = add_item("apple")
print(a)  # ['apple']

b = add_item("banana")
print(b)  # ['apple', 'banana']

c = add_item("cherry")
print(c)  # ['apple', 'banana', 'cherry']
```

코드만 보면 add_item 함수의 파라미터 item_list는 빈 리스트로 초기화되고 있다. 그런데 막상 이 코드를 실행하면 item_list의 값이 누적되는 현상을 볼 수 있다.

## 원인

결론은 **기본 인자의 값은 함수가 정의될 때 생성되고 함수 객체에 바인딩되며, 호출할 때는 다시 생성되지 않는다.** 그래서 아래와 같이 item_list의 id를 출력해보면 실제로를 매번 같은 객체를 사용하고 있음을 확인 할 수 있다.

```python
def add_item(item, item_list=[]):
    print(f"item_list의 id: {id(item_list)}")
    item_list.append(item)
    return item_list

add_item("apple")   # id: 140234567890
add_item("banana")  # id: 140234567890  <- 같은 객체!
add_item("cherry")  # id: 140234567890
```

## 언제 이런 상황이 발생할까

파라미터 기본값이 한 번만 생성되는 부분은 잊지 말자. 그러므로 Mutable 객체(리스트, 딕셔너리, 세트 등)는 값이 변경될 수 있기에 Mutable 객체를 기본값으로 사용할 때 문제가 된다.

```python
# 문제가 되는 경우
def add_user(name, user_dict={}):           # 딕셔너리
    user_dict[name] = len(user_dict) + 1
    return user_dict

def add_tag(tag, tags=set()):               # 세트
    tags.add(tag)
    return tags

# 문제가 안 되는 경우
def greet(name, greeting="Hello"):          # 문자열 (immutable)
    return f"{greeting}, {name}!"

def calculate(x, multiplier=2):             # 숫자 (immutable)
    return x * multiplier
```

## 해결 방법

리서칭한 결과 **None을 기본값으로 사용**하는 방법이 가장 권장되는 방식이다. 그리고 혹시라도 함수안에서 의도적으로 이렇게 상태를 유지하고 싶다면 상태를 저장하는 클래스나 전역 변수를 사용하는게 더 명확하다.

```python
def add_item(item, item_list=None):
    if item_list is None:
        item_list = []
    item_list.append(item)
    return item_list
```
