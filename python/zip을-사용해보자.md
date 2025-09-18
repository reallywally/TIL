# zip을 사용해보자

## 개요

zip은 여러 이터러블(리스트, 튜플 등)을 병렬로 묵어 주는 내장함수이다.

## 기본 사용법

```python
a = [1, 2, 3]
b = ["a", "b", "c"]
z = zip(a, b)

print(list(z))  # [(1, 'a'), (2, 'b'), (3, 'c')]
```

## 중요한 성질들

- zip은 이터레이터로 반환하여 한번 반복문을 돌면 소진된다.

    ```python
    a = [1, 2, 3]
    b = ["a", "b", "c"]

    z = zip(a, b)

    print(list(z))  # [(1, 'a'), (2, 'b'), (3, 'c')]

    print(list(z))  # [] <-- 소진되어 없음

    ```

- 인덱스로 접근이 불가능하다. 필요하면 list()나 tuple()로 감싸야 한다.

    ```python
    a = [1, 2, 3]
    b = ["a", "b", "c"]

    z = zip(a, b)

    print(z[0])  # 에러 

    print(list(z[0]))  # (1, "a")
    ```

- 서로 다른 길이의 이터러블이면 짧은 쪽에 맞춰 멈춘다

    ```python
    x = [1, 2, 3]
    y = ["a", "b"]
    z = zip(x, y)

    print(list(z))  # [(1, 'a'), (2, 'b')] -> 세번째 요소는 짤린다.
    ```

## 자주 쓰이는 패턴 / 실무 예쩨

1. 병렬 반복

    ```python
    for x, y in zip(list1, list2):
        # list1과 list2를 같이 반복
    ```

2. 행렬 전치

    ```python
    matrix = [[1,2,3], [4,5,6], [7,8,9]]
    cols = list(zip(*matrix))    # [(1,4,7),(2,5,8),(3,6,9)]
    ```

3. 헤더와 데이터를 행으로 합쳐서 딕셔너리 만들기

    ```python
    headers = ["name", "age", "city"]
    data = ["alice", 30, "서울"]

    d = dict(zip(headers, data))  # {'name':'Alice','age':30,'city':'Seoul'}
    ```
