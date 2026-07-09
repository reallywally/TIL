# Item 34 *args로 시각적 노이즈를 줄여라

## 시작: 고정 인자의 불편함

흔히 varargs, star args라 부르고, 관례적 이름이 *args라는 가변 위치 인자를 사용하면 함수 호출이 더 깔끔해질 수 있다. 디버깅 정보를 로깅하는 함수를 생각해보자. 고정 인자를 사용하면  메시지와 값 리스트를 받아야 한다.

```python
def log(message, values):
    if not values:
        print(message)
    else:
        values_str = ", ".join(str(x) for x in values)
        print(f"{message}: {values_str}")

log("숫자들:", [1, 2])
log("안녕", [])            # 값이 없어도 빈 리스트를 넘겨야 함 (번거로움)
```

동작에는 문제가 없지만 값이 없을 때도 빈 리스트 []를 넘겨야 해서 번거롭다.(기본값처리도 가능하지만 예시니까 넘어가자)

## *args로 개선

마지막 위치 매개변수 이름 앞에 *를 붙이면, 첫 인자는 필수이고 그 뒤 위치 인자는 몇 개든 선택이 된다. 함수 본문은 그대로고 호출부만 깔끔해진다.

```python
def log(message, *values):    # 변경
    if not values:
        print(message)
    else:
        values_str = ", ".join(str(x) for x in values)
        print(f"{message}: {values_str}")

log("숫자들:", 1, 2)
log("안녕")                 # 빈 리스트 불필요!
```

## 문제 1: *args는 항상 튜플로 만들어진다

선택 위치 인자들은 함수에 전달되기 전에 **항상 튜플로 변환**된다. 그래서 호출자가 **제너레이터에 *를 쓰면 끝까지 소진되어** 모든 값이 튜플에 담기고 값이 많으면 메모리를 크게 잡아먹어 프로그램이 죽을 수 있다.  
따라서*args는 **입력 개수가 충분히 적다고 확신할 때** 사용하는것이 적합하다. 여러 리터럴이나 변수명을 함께 넘기는 호출에 이상적이고, 주로 **호출자의 편의와 호출 코드의 가독성**을 위한 기능이다.

## 문제 2: 인자 변경의 유연함으로 발생한 버그

*args는 앞서 설명한것과 같이 마지막 인자를 튜플로 받는다. 그러다 보니 인자를 잘못 입력하여도 함수 호출이 되어 예상하지 못한 버그가 발생할 수 있다.

```python
def log_seq(sequence, message, *values):
    if not values:
        print(f"{sequence} - {message}")
    else:
        values_str = ", ".join(str(x) for x in values)
        print(f"{sequence} - {message}: {values_str}")

log_seq(1, "Favorites", 7, 33)      # 정상
log_seq("Favorite numbers", 7, 33)  # 비정상 — 7이 message로 들어가 버림!
```

예시 코드를 보면 첫번째 함수 호출 처럼 의도한대로 인자가 입력되면 상관 없다. 그런데 두번째 함수 호출은 sequence 인자를 안 줬기 때문에 7이 message로 해석된다. 예외 처리도 안되어 있어 이런 버그는 추적하기 매우 어렵다.  
이를 완전히 피하려면 *args를 받는 함수를 확장할 때 **키워드 전용 인자(keyword-only, Item 37)**를 쓰고, 더 방어적으로 하려면 타입 애너테이션도 고려해야한다.
