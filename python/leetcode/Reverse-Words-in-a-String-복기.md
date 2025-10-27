# Reverse Words in a String 복기

## 내가 만든 코드

```python
class Solution:
    def reverseWords(self, s: str) -> str:
        trim_str = s.strip()
        split_str = trim_str.split(" ")
        split_str.reverse()
        split_str = [s for s in split_str if s != ""]

        result = " ".join(split_str)

        return result
```

## 베스트 정답

```python
class Solution:
    def reverseWords(self, s: str) -> str:
        return ' '.join(s.split()[::-1])
```

## 오답노트

### 1. 불필요한 중간 변수

각 단계마다 새로운 변수를 만들어 불필요한 메모리를 추가로 사용하였다.

```python
    split_str = trim_str.split(" ")
    split_str.reverse()
    split_str = [s for s in split_str if s != ""]
```

### 2. split(" ") VS split()

이번 문제의 핵심이다. 특정 문자로 분리하기 위해 ```split()```를 사용했고 공란으로 분리하기 위해 ```split(" ")```로 하였지만 사실 ```split()```의 기본 특성이 **자동으로 모든 공백, 빈 문자열**을 처리한다.

```python
split_str = trim_str.split(" ")  # 내 코드
# 결과: ["hello", "", "", "world"] (여러 공백 시)

s.split()  # 최적화된 방법
# 결과: ["hello", "world"] (자동으로 모든 공백 처리)
```

### 3. 추가 필터링 작업

2번의 ```split()```을 잘 사용했다면 발생하지 않았을 코드이다.

```python
split_str = [s for s in split_str if s != ""]  # 불필요 했던 코드
```

### 4. reverse() VS [::-1]

```python
split_str.reverse()  # in-place, 원본 수정
split_str[::-1]      # 새 리스트 생성, 하지만 더 빠름
```

파이썬에서는 슬라이싱이 C 레벨에 최적화되어 있어 파이썬 함수 보다 더 빠르다.

## 핵심 포인트

- split()의 특성
- 슬라이싱 속도

## 후기

문제만 보고 쉬운것 같아 생각나는데로 코딩했더니 문제는 풀었지만 코드 퀄리티가 좋진 않았다. 사실 고민한다고 해도 좀 더 파이썬 스럽게 만들지는 못 했을거 같다.
