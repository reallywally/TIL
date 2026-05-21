# item9 구조 분해에는 match를 고려하되, if문으로 충분할 땐 피하라

## match 기본

if문으로 힘들게 표현하는 코드를 match로 수정하면 가독성을 향상 시킬 수 있다.

```python
# if 버전 (9줄)
def take_action(light):
    if light == "red":
        print("Stop")
    elif light == "yellow":
        print("Slow down")
    elif light == "green":
        print("Go!")
    else:
        raise RuntimeError

# match 버전 (10줄):
def take_match_action(light):
    match light:
        case "red":
            print("Stop")
        case "yellow":
            print("Slow down")
        case "green":
            print("Go!")
        case _:          # else에 해당하는 와일드카드
            raise RuntimeError
```

그런데 가독성은 좋아젔지만 match를 쓰고 싶을 만큼 큰 장점은 안보인다.

## match의 핵심 함정: 캡처 패턴

match의 최대 장점을 느끼기 위해 match를 조금만 더 알아보자. java의 switch문을 생각하면 match의 case 또한 값 비교처럼 보인다.
하지만 match의 case는 **비교가 아니라 할당**이다. 그래서 아래 예시 코드의 ```case RED:``` 부분이 ```light == RED```가 아니라 ```(RED,) = (light,)```로 동작한다. 그래서 파라미터로 RED가 아닌 YELLOW 넣으면 "slow down"이 출력되지 않고 "SyntaxError: name capture 'RED' makes remaining patterns unreachable" 이런 에러가 출력된다.

```python
RED = "red"
YELLOW = "yellow"
GREEN = "green"

def take_constant_action(light):
    match light:
        case RED:    # ==이 아님!
            print("Stop")
        case YELLOW:
            print("Slow down")

```

그래서 위 코드가 정상적으로 동작할려면 점(.) 연산자로 상수 취급할수 있게 해야하고 enum을 쓰면 된다.

```python
import enum

class ColorEnum(enum.Enum):
    RED = "red"
    YELLOW = "yellow"
    GREEN = "green"

def take_enum_action(light):
    match light:
        case ColorEnum.RED:      # .이 있으므로 값 비교 ✅
            print("Stop")
        case ColorEnum.YELLOW:
            print("Slow down")
        case ColorEnum.GREEN:
            print("Go!")
        case _:
            raise RuntimeError
```

## match가 진짜 빛나는 곳 — 구조 분해(Destructuring)

match의 존재 이유는 복잡한 중첩 데이터를 패턴으로 처리할 수 있는것이다.
책은 예시로 재귀함수를 사용했는데 이해가 어플리케이션 개발자에게는 친숙하지 않은 예시같아 dictionary만 작성했다.
나중에 실무에서 match를 활용해서 문제를 해결하게되면 수정하자.
암튼 API 응답이 왔다 가정하고 response를 다음과 같이 match로 처리할 수도 있다. 만약 ```data["hello"]```값이 "Hello, World!!"이면 222이 출력된다.

```python
response = {"data": {"hello": "Hello, World!"}, "msg": "success"}


def test_response_match(res: dict[str, dict[str, str] | str]):
    match res:
        case {"data": {"hello": "Hello, World!"}, "msg": "success"}:
            print("111")
        case {"data": {"hello": "Hello, World!!"}, "msg2": "success"}:
            print("222")
        case _:
            assert False, "Response did not match expected pattern"


test_response_match(response)
# 111
```

## 한줄 요약

match로 단순 if를 대체하는기 보단 진짜 강점인 isinstance 체크와 구조 분해를 흐름 제어와 결합하여 활용하자.
