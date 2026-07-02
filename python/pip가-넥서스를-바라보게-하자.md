# pip가 넥서스를 바라보게 하자

## 상황

내부망은 외부와 단절되어 필요한 패키지를 넥서스로 관리한다. 그런데 pip가 똑똑해서 자동으로 내부 넥서스를 바라보지 못하기 때문에 설정을 해주어야 한다.

## 해결방법

### 1. 명령어에서 일회성으로 지정

권장되는 방식은 아니지만 이런 방법도 있다고는 알고 있자

```bash
pip install requests \
  --index-url http://nexus.mycompany.com:8081/repository/pypi-proxy/simple \
  --trusted-host nexus.mycompany.com
```

### 2. 글로벌 설정

일반적으로 권장되는 방식이다. 한번 설정하면 두번 설정하지 않아서 좋다.

```cmd
pip config set global.index-url http://nexus.mycompany.com:8081/repository/pypi-proxy/simple
pip config set global.trusted-host nexus.mycompany.com
```

### 3. 환경변수로 지정

Docker 빌드나 Kubernetes 환경처럼 설정 파일 두기 애매한 경우에 유용하다.

```bash
export PIP_INDEX_URL=http://nexus.mycompany.com:8081/repository/pypi-proxy/simple
export PIP_TRUSTED_HOST=nexus.mycompany.com
```

## 주의사항

- 예시 URL에서 /repository/pypi-proxy/simple의 pypi-proxy는 넥서스에서 관리하는 Browse 이름에 따라 달라지니 잘 확인 하자.
