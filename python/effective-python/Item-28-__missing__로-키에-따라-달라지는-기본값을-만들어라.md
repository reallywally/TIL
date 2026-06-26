# Item 28 __missing__로 키에 따라 달라지는 기본값을 만들어라

지금까지 setdefault(Item 26)와 defaultdict(Item 27)를 봤는데, 둘 다 안 맞는 상황이 있다. 바로 기본값을 만드는 비용이 크거나, 예외가 날 수 있거나, 기본값이 키에 따라 달라져야 할 때이다.

## 문제 상황: 파일 경로 → 열린 파일 핸들 매핑

소셜 네트워크 프로필 사진을 파일 시스템에서 관리한다고 가정하자. 이때 경로를 key로 열린 파일 핸들(파일에 대한 작업 통로)을 가저오는 딕셔너리가 필요하다. 일반 dict + get + 대입 표현식으로 코드를 만들면 다음과 같다.

```python
pictures = {}
path = "profile_1234.png"

if (handle := pictures.get(path)) is None:
    try:
        handle = open(path, "a+b")
    except OSError:
        print(f"경로 열기 실패: {path}")
        raise
    else:
        pictures[path] = handle

handle.seek(0)
image_data = handle.read()
```

코드는 나름 직관적이다. 딕셔너리에 값이 있으면 바로 사용하고, 없으면 기본값을 설정하면서 예외처리도 한다. 그런데 라인이 좀 길어서 다소 장황하다.

## setdefault로는 안 되는 이유

```python
try:
    handle = pictures.setdefault(path, open(path, "a+b"))
except OSError:
    ...
```

문제투성이다. open이 경로가 이미 있든 없든 항상 호출되어, 같은 프로그램의 기존 핸들과 충돌할 수 있는 여분의 파일 핸들이 생긴다. 또한 open이 던지는 예외와 setdefault(같은 줄) 자체가 던질 수 있는 예외를 구분하기 어렵다.

## defaultdict로도 안 되는 이유

```python
from collections import defaultdict

def open_picture(profile_path):
    try:
        return open(profile_path, "a+b")
    except OSError:
        print(f"경로 열기 실패: {profile_path}")
        raise

pictures = defaultdict(open_picture)
handle = pictures[path]
# TypeError: open_picture() missing 1 required positional argument: 'profile_path'
```

defaultdict에 넘기는 함수는 **인자를 받지 않아야 한다**. 즉, 기본값을 만드는 함수가 어떤 키에 접근 중인지 알 수 없다. 그래서 open(path)처럼 키를 써야 하는 경우엔 무용지물이다.

## 해결책: dict를 상속하고 ```__missing__``` 구현

dict를 상속해 ```__missing__``` 특수 메서드를 구현하면, 누락된 키 처리에 커스텀 로직을 넣을 수 있다. 위에서 만든 open_picture 헬퍼를 그대로 활용하면 다음과 같다.

```python
class Pictures(dict):
    def __missing__(self, key):
        value = open_picture(key)   # 키(경로)를 알고 있으니 open 가능!
        self[key] = value           # 직접 딕셔너리에 저장
        return value                # 호출자에게 반환

pictures = Pictures()
handle = pictures[path]
handle.seek(0)
image_data = handle.read()
```

pictures[path] 접근 시 키가 없으면 __missing__이 호출된다. 이 메서드는 기본값을 만들고, 딕셔너리에 넣고, 반환해야 한다. 같은 path에 다시 접근하면 이미 항목이 있으므로 __missing__은 호출되지 않는다. 이렇게 하면 open은 키가 없을 때 딱 한 번만 호출되고, 키를 알고 있으니 키 의존적 기본값도 만들 수 있다.

## 결과는 똑같잖아?

사실 처음 문제 상황에서 제시한 대로 코드를 만들어도 동작은 한다. 초급자(과거의 나) 관점에서는 `__missing__`를 구현하는 것만 다르지, 전체 코드로 보면 다 똑같은데 뭐가 다른지 이해가 안 될 수도 있다.

차이는 **"키가 있는지 확인하고, 없으면 값을 만들어 저장하는" 로직을 `__missing__` 한곳에 모을 수 있다는 점**이다. 이렇게 하면 딕셔너리를 사용하는 쪽은 그냥 `pictures[path]`처럼 접근만 하면 되고, "처음 접근이면 만들고 아니면 재사용"하는 복잡함을 신경 쓸 필요가 없다. 만약 처음 코드대로 했다면, 이 딕셔너리를 쓰는 모든 곳에서 매번 그 분기 로직을 반복해야 했을 것이다.

게다가 `__missing__` 방식은 **이미 핸들이 있을 땐 `open`을 다시 호출하지 않는다.** 그래서 같은 파일에 대해 불필요한 핸들이 중복으로 열리는 일을 막아준다. (`setdefault`로 짰다면 키가 이미 있어도 `open`이 매번 호출되어 이 문제가 생긴다.)

## 한줄 요약

dict의 setdefault는 기본값 생성 비용이 크거나 예외가 날 수 있을 때 부적합하고 이럴때 ```__missing__``` 메서드를 가진 dict 서브클래스를 정의하자.
