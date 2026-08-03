# @staticmethod vs @classmethod. 언제 뭘 써야할까?

2개의 차이점을 잘 알고 있었다 생각했는데 막상 다른 사람한테 설명하려 하니 좀 어려웠다. 그래서 다시 한번 명확하게 개념을 정리하고자 글을 작성한다.

## 코드의 차이

코드로 보면 우선 암묵적으로 전달되는 첫 번째 인자이다. 이 인자는 이름이 고정은 아니지만 관용적으로 인스턴스 메소드는 self, 클래스 메소드는 cls를 사용한다. 반면 static 메소드는 이런 바인딩 인자를 받지 않는다. 일반 인자는 얼마든지 받을 수 있으니 "인자가 아예 없다"와는 다르다.

```python
class MyClass:
    class_var = "클래스 변수"
    
    def instance_method(self):
        # self: 인스턴스 자신
        return f"인스턴스 메서드, {self}"
    
    @classmethod
    def class_method(cls):
        # cls: 클래스 자신
        return f"클래스 메서드, {cls.class_var}"
    
    @staticmethod
    def static_method():
        # 아무것도 받지 않음
        return "정적 메서드"
```

## 사용의 차이

```@staticmethod```는 클래스나 인스턴스 정보가 필요 없는 독립적인 함수다. 그래서 인스턴스 메소드나 클래스 메소드와 다르게 **self나 cls 같은 바인딩 인자를 받지 않는다.** 사실상 일반 함수와 동일하나 네임스페이스가 클래스 안에 있는것이다. 대표적인 예시로 유틸 클래스가 있다.

```python
class StringUtils:
    @staticmethod
    def is_palindrome(text):
        # 클래스 정보가 필요 없음
        clean = text.replace(" ", "").lower()
        return clean == clean[::-1]
    
    @staticmethod
    def count_vowels(text):
        vowels = 'aeiou'
        return sum(1 for char in text.lower() if char in vowels)

# 사용
print(StringUtils.is_palindrome("A man a plan a canal Panama"))  # True
print(StringUtils.count_vowels("Hello World"))  # 3
```

이렇게 논리적으로 문자열 관련 유틸리티 기능을 하는 클래스이지만 메소드가 클래스 상태에 접근할 필요가 없는 경우에 ```@staticmethod```를 붙인다.

반면 ```@classmethod```는 클래스 자체를 첫 번째 인자(cls)로 받는다. 이 cls를 통해 클래스 변수에 접근하거나, 클래스의 새 인스턴스를 생성할 수 있다.

```python
class MyClass:
    class_variable = "나는 클래스 변수"
    
    @classmethod
    def show_class_info(cls):
        # cls는 MyClass를 가리킴
        print(f"클래스 이름: {cls.__name__}")
        print(f"클래스 변수: {cls.class_variable}")
        
        # cls로 새 인스턴스 생성 가능
        new_instance = cls()
        return new_instance

MyClass.show_class_info()
# 클래스 이름: MyClass
# 클래스 변수: 나는 클래스 변수
```

여기서 짚고 갈 점 하나. ```@staticmethod```와 ```@classmethod```는 **인스턴스에서도 호출할 수 있다.** 위 예시들이 전부 클래스로 호출하고 있어서 "클래스로만 부르는 것"으로 오해하기 쉬운데 그렇지 않다.

```python
MyClass().show_class_info()          # 동작함. cls는 여전히 MyClass
StringUtils().count_vowels("Hello")  # 동작함
```

## @staticmethod를 안 붙이면?

그럼 클래스 안에 그냥 함수를 두는 것과 뭐가 다를까. 데코레이터가 없으면 인스턴스로 호출할 때 self가 자동으로 첫 번째 인자에 들어간다.

```python
class Foo:
    def plain(text):          # 데코레이터 없음
        return text.upper()

    @staticmethod
    def static(text):
        return text.upper()

Foo.plain("hi")     # "HI"  - 클래스로 부르면 문제없음
Foo().plain("hi")   # TypeError: Foo.plain() takes 1 positional argument but 2 were given
Foo().static("hi")  # "HI"
```

즉 ```@staticmethod```는 **이 자동 바인딩을 끄는 역할**이다. 클래스로만 부를 거면 티가 안 나지만, 인스턴스로 부르는 순간 차이가 드러난다.

## 그래서 언제 뭘 써야 하나

결정적인 기준은 **상속**이다. ```cls```는 실제로 호출된 클래스에 바인딩되지만, ```@staticmethod``` 안에서는 클래스 이름을 직접 써야 하므로 그 클래스로 고정된다.

```python
class Animal:
    @classmethod
    def create(cls):
        return cls()           # 실제 호출된 클래스로 바인딩

    @staticmethod
    def create_static():
        return Animal()        # Animal로 고정

class Dog(Animal):
    pass

print(type(Dog.create()))         # <class 'Dog'>
print(type(Dog.create_static()))  # <class 'Animal'>  ← 의도와 다름
```

서브클래스에서 다르게 동작해야 하면 ```@classmethod```, 아니면 ```@staticmethod```.

## 팩토리 메소드 패턴

```@classmethod```의 실무 용례 1순위. ```__init__``` 말고 다른 방식으로 인스턴스를 만들고 싶을 때 쓴다. 대체 생성자(alternative constructor)라고도 부른다.

```python
class Date:
    def __init__(self, year, month, day):
        self.year, self.month, self.day = year, month, day

    @classmethod
    def from_string(cls, date_str):
        return cls(*map(int, date_str.split("-")))

Date.from_string("2026-07-31")
```

표준 라이브러리에서도 흔하다. ```dict.fromkeys()```, ```datetime.now()```, ```datetime.fromtimestamp()``` 모두 이 패턴이다.

## 요약

| 필요한 것 | 선택 |
|---|---|
| 인스턴스 상태(self) 접근 | 인스턴스 메소드 |
| 클래스 자체가 필요 (팩토리, 상속에 따라 달라짐) | ```@classmethod``` |
| 아무것도 필요 없음, 논리적으로만 클래스에 속함 | ```@staticmethod``` |
