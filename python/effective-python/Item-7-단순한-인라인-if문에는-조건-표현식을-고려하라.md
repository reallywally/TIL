# Item7 단순한 인라인 if문에는 조건 표현식을 고려하라

## 기본 문법

python의 인라인 if문 아래와 같은 문법을 가진다.

```python
i = 3
x = "even" if i % 2 == 0 else "odd"  # 참값 if 조건 else 거짓값
# "odd"
```

## 삼항 연산자랑은 다르다

c나 java에서도 비슷하게 삼항 연산자를 많이 사용한다. 하지만 문법이 다르니 헷갈리지 말자.

- C/Java: 조건 ? 참값 : 거짓값
- Python: 참값 if 조건 else 거짓값

## 인라인 if문을 사용하면 좋은 경우

조건이 단순하고 한 줄에 딱 들어올 때 쓰면 매우 좋다.

```python
x = "even" if i % 2 == 0 else "odd"  # ✅ 명확하고 짧음
```

## 일반 if문을 사용하면 좋은 경우

나중에 확장 가능성이 있다면 일반 if문을 사용하는게 좋다.

```python
if i % 2 == 0:
    x = "even"
    print("It was even!")  # 디버깅 로그 추가가 자연스러움
else:
    x = "odd"
```

## 인라인 if문을 피해야 하는 경우

한줄로 코드를 작성하는게 무조건 좋은게 아니다. 읽기 힘들고 auto-formatter가 더 길게 만들어버리면 분리하자.

```python
# 나쁨
x = (my_long_function_call(1, 2, 3)
     if i % 2 == 0
     else my_other_long_function_call(4, 5, 6))

# 수정
if i % 2 == 0:
    my_long_function_call(1, 2, 3)
else:
    my_other_long_function_call(4, 5, 6)

```

## 한줄 요약

인라인 if는 만능이 아니고, 상황에 맞게 판단해야 한다.
