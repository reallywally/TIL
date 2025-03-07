# API Mocking을 사용해볼까

## 왜 쓰는 걸까

프론트 작업을 하는데 서버가 개발이 안된 경우 화면에 표시할 데이터가 없다.  
이때 가짜 데이터가 필요한데 수동으로 관리하면 번거로운 부분이 많아 API Mocking 라이브러리를 이용하면 관리가 편할것 같다.

## 무엇을 사용할까

여러가지가 있는데 아래 2가지를 많이 쓰는것 같다.

1. Mock Service Worker(MSW)
2. JSON Server

## 실제로 사용해 보니

MSW, JSON Server 두개 다 사용해 봤는데 내가 원하는대로 사용할 수 없었다.
나는 기본적으로 개발서버가 있으니 지금 개발 중인 화면에 필요한 API만 Mocking을 하고 싶었다.
그런데 MSW는 코드 레벨에서 관리가 필요해서 전체 적용이 되어 지금 시스템에서는 오버 스펙이었고 JSON Server는 요청에 따라 응답 객체를 커스텀하는 작업이 어려웠다.
결국 마음에 드는건 없었다.

## 그래서 내가 사용한 방법은

로컬에서 flask로 응답만 하는 모킹 서버를 만들어 사용하고 있다.
flask 자체가 굉장히 간단하게 서버를 실행할 수 있고 파이썬이 JSON을 관리하는데 편리하다.
협업 관점에서는 MSW가 좋아보이나 지금은 간단한 사용만 필요하였다.
디렉토리는 특별히 관리하지 않고 필요한것만 그때그때 수정하는 식으로 하니 지금까진 불편한 사항은 없다.

### flask 샘플 코드

- 디렉토리

  ```text
    project
      -json
        - userList.json
        - userDetail.json
      -app.py
  ```

- flask code

  ``` python
  from flask import Flask, jsonify
  import json

  app = Flask(__name__)


  def read_json_file(file_path):
      with open(file_path, 'r') as file:
          data = json.load(file)

      return data


  @app.get('/users')
  def user():
      data = read_json_file("json/userList.json")
      return jsonify(data)


  @app.get('/users/<string:user_id>')
  def user(user_id):
      data = read_json_file("json/userDetail.json")
      return jsonify(data)

  ```

## 메모

- MSW는 1.x에서 2.x로 바뀌면서 많은 부분이 변경된거 같다. 잘 찾아보고 하자
  - <https://mswjs.io/docs/migrations/1.x-to-2.x/>
- JSON Server를 사용할때 GET users/1 이라는 요청에 응답할려면 JSON 키로 users/1이 아니라 아래와 같이 목록으로 저장하면 id로 조회한다.

  ```json
  {
    "users": [
      {
        "id": "1",
        "name": "John",
        "age": 25
      },
      {
        "id": "2",
        "name": "Doe",
        "age": 30
      }
    ],
  }
  ```
