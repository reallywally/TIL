# flask api요청을 DTO로 변경해보자

## 배경

스프링은 DTO 클래스를 만들면 API 요청을 받을때 필드 이름에 맞춰 맵핑이 되기 때문에 사용이 간단하다. 하지만 flask는 ```request.get_json()```으로 받아 딕셔너리로 처리를 해야한다. 당연히 딕셔너리로 처리하다보니 휴먼 에러의 가능성이 높고 벨리데이션이 어려워지는 문제가 발생한다. 이를 해결할 수 있는 방법을 찾아보자.

## 해결 방법들

1. pydantic  
리서칭 해보면 가장 많이 나오는 방법이다. 사용법도 간단하고 좋아 보인다. 하지만 안타깝게도 하이퍼커넥트에서 만든 자료를 보면 속도가 많이 느리다고 한다. 지금 개발하는 시스템은 요청 사이즈가 크지 않아 이슈가 없을 것으로 예산하지만 성능 이슈는 항상 마음에 걸린다.

2. marshmallow

3. 파이썬 dataclass  

## 결론

## 참고자료

* 하이퍼커넥트 Pydantic은 아주 느리다:  <https://hyperconnect.github.io/2023/05/30/Python-Performance-Tips.html#5-pydantic%EC%9D%80-%EC%95%84%EC%A3%BC-%EB%8A%90%EB%A6%AC%EB%8B%A4-%EB%B6%88%ED%95%84%EC%9A%94%ED%95%9C-%EA%B3%B3%EC%97%90%EC%84%9C-%EA%B0%80%EA%B8%89%EC%A0%81-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EB%A7%90%EC%9E%90>
* 스포카 마시맬로 사용기: <https://spoqa.github.io/2021/03/23/flask-marshmallow-apispec.html>
