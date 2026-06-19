# Item 4 복잡한 표현식 대신 헬퍼 함수를 써라

## 예제 상황: URL 쿼리 스트링 파싱

URL에서 ```red=5&blue=0&green=``` 같은 파라미터를 파싱한다고 가정해보자.

```python
from urllib.parse import parse_qs

my_values = parse_qs("red=5&blue=0&green=", keep_blank_values=True)
# {'red': ['5'], 'blue': ['0'], 'green': ['']}
```

red는 '5', blue는 '0', green은 빈 문자열 '', opacity는 키 자체가 없음(None). 세 가지 경우가 섞여 있다.

## 1단계: 한 줄로 욱여넣기 (나쁜 예)

"값이 없으면 0으로 처리하고 싶다"는 요구사항을 한 줄로 쓰면 다음과 같다.

```python
red = my_values.get("red", [""])[0] or 0
```

코드 라인을 줄이고 한줄로 처리 했다는 뿌듯?함이 생길 수 있다.  근데 처음 보는 사람한테 설명하려면 한참 걸린다. 여기에 int() 변환까지 추가하면 더 읽기 힘들어 진다.

```python
red = int(my_values.get("red", [""])[0] or 0)
```

그리고 파라미터는 red 1개가 아니다. green, opacity까지 포함하면 총 세 개에 똑같은 로직을 반복해야 하고 파라미터가 증가할 수록 유지보수는 더 힘들어 진다.

## 2단계: if문으로 풀기 (중간 단계)

코드를 조금 더 가독성 좋게 만들어 보자.

```python
green_str = my_values.get("green", [""])
if green_str[0]:
    green = int(green_str[0])
else:
    green = 0
```

한 줄보다는 낫지만, 이걸 3개 변수에 반복하면 코드가 엄청 길어진다.

## 3단계: 헬퍼 함수로 뽑기 (정답)

그렇다면 가장 베스트 방법은 무엇을까. 제목에서 언급한대로 **헬퍼 함수**로 분리하는것이다.

```python
def get_first_int(values, key, default=0):
    found = values.get(key, [""])
    if found[0]:
        return int(found[0])
    return default

pythonred   = get_first_int(my_values, "red")
green = get_first_int(my_values, "green")
opacity = get_first_int(my_values, "opacity")
```

함수 이름만 봐도 뭘 하는지 바로 이해된다. DRY 원칙 (Don't Repeat Yourself) 도 지켜고 아주 좋다.

## 사실 헬퍼 함수는 python에 국한된건 아니다

어떤 언어로 작성하든 1단계, 2단계로 코드를 작성하는건 좋은 방법이 아니다. 하지만 python에서 좀 더 강조하는 이유는 문법적으로 허용되기 때문이다. 마치 한 줄에 모든걸 해결하면 개발을 잘 하는것 같은 착각 하기 쉽다. 문법이 허용된다 한들 **클린코드**의 관점을 잊지 말아야 한다.

## 한줄 요약

python은 문법이 간결해서 한 줄에 많은 걸 욱여넣기가 너무 쉬운데 오히려 이게 독이 될수 있으니 조심하자.
