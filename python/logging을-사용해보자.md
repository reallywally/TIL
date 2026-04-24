# logging을 사용해 보자

## 1. 로깅 라이브러리 사용

structlog , Loguru 등 로깅 라이브러리가 있다. 자세한 사용법은 이 글에서 중요하지 않으니 생략한다.

## 2. Mixin 패턴

아래와 같이 클래스를 만들어 상속 받아 처리할 수도 있다. 다만 상속으로 처리되기 때문에 아래 예시 처럼 단순 구조를 잘 사용하지 않고 클래스 설계를 잘 해야될것 같다.

```python
import logging

class LoggingMixin:
    @property
    def logger(self):
        # 클래스의 실제 경로를 포함한 이름을 로거명으로 사용
        name = f"{self.__module__}.{self.__class__.__name__}"
        return logging.getLogger(name)

class UserService(LoggingMixin):
    def create_user(self):
        self.logger.info("사용자 생성 시작")
```

## 3. 데코레이터

아래와 같이 데코레이터를 만들고 붙여서 사용할 수 있다. 스프링 개발자에게 익숙한 형태이다. 단점은 클래스에 데코레이터를 붙여야 사용이 가능하다.  
자바는 기능을 개발할때 모두 클래스 안에 함수를 만들어서 어색하지 않지만 파이썬의 경우 파일에 함수만 만드는 경우도 많기 때문에 이 방법은 파이썬스럽지 않은 느낌을 준다.

```python
def add_logger(cls):
    cls.logger = logging.getLogger(f"{cls.__module__}.{cls.__name__}")
    return cls

@add_logger
class UserService:
    def save(self):
        self.logger.info("저장 중...")
```

## 4. 표준 방식😊

결국 가장 표준 방식을 사용하는게 좋다. 클로드가 말하길 파이썬 생태계 전반에서 쓰는 방식이라한다. __name__으로 모듈 경로가 자동으로 찍혀서 어디서 난 로그인지 바로 알 수 있는 장점이 있다.  

```python
import logging
logger = logging.getLogger(__name__)

logger.info("로그!")
```

## 여담

표준 방식이 좋은건 알겠는데 처음엔 스프링과 비교했을때 에노테이션으로 처리하는게 아니어서 조금 어색했다.
