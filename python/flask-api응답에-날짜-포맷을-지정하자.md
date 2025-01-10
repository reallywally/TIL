# Flask API응답에 날짜 포맷을 지정하자

## 상황

ORM 모델에 type이 Date인 필드가 있다. 디버깅 해보면 API return 전까지만해도 ```YYYY-MM-DD```형태로 보여서 문제가 없을것 같았는데 실제로 응답은 ```Fri, 01 Jan 2021 00:00:00 GMT```형태로 리턴되었다.

## 해결 방법

1. 수동 포맷팅  
    리턴 할때마다 날짜 변수를 isoformat로 변환해서 리턴하도록한다. 하지만 매번 하기도 번거롭고 휴먼 에러의 가능성이 매우 높기 때문에 글로벌 설정이 필요하다.

    ```python
    def test_response():
        result = {
            "status": 200, 
            "is_return_snake": date.isoformat()
        }

        return jsonify(result)
    ```

2. 글로벌 설정  
    flask 앱을 초기화 할때 JSON Provider를 생성해서 넣어준다.

    ```python
    from datetime import datetime, date
    from flask.json.provider import DefaultJSONProvider


    class CustomJSONProvider(DefaultJSONProvider):
        def default(self, obj):
            if isinstance(obj, datetime):
                return obj.strftime('%Y-%m-%d %H:%M:%S')
            if isinstance(obj, date):
                return obj.isoformat()
            return super().default(obj)


    app = Flask(__name__)
    app.json = CustomJSONProvider(app)
    ```

## etc

Flask 버전 3.X를 기준으로 작성했다. 이전 버전에는 import하는 클래스랑 app에 넣는 필드가 다르니 잘 찾아보자. 그리고 정상적인 코드는 ```app.json = CustomJSONProvider(app)```인데 GPT씨가 대신 ```app.json = CustomJSONProvider```로 알려줘서 삽질을 많이 했다. 다행히 영어로 물어보니 똑바로 대답했다.
