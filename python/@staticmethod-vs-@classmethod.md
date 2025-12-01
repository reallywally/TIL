# @staticmethod vs @classmethod. 언제 뭘 써야할까?

## 시작

2개의 차이점을 잘 알고 있었다 생각했는데 막상 다른 사람한테 설명하려 하니 좀 어려웠다. 그래서 다시 한번 명확하게 개념을 정리하고자 글을 작성한다.

## 표면적 차이

코드로 보면 우선 첫 번째 인자값이 이다.

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

```@staticmethod```는 클래스나 인스턴스 정보가 필요 없는 독립적인 함수다. 사실상 일반 함수와 동일하나 네임스페이스가 클래스 안에 있는것이다. 대표적인 예시로 유틸 클래스가 있다.

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

반면 ```@classmethod```