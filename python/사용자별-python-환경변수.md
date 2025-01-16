# 사용자별 python 환경변수

## 상황

우분투 서버에서 파이썬 패키지를 관리하던 중 이상한? 상황이 생겼다. 일반사용자로 ```python```을 실행하면 실행되는데 ```sudo python``` 하면 파이썬을 못찾았다. 항상 venv 가상 환경을 사용해서 이런 경우가 없었던지라 더 당황했다.

## PATH 환경 변수 차이

결론적으로 **사용자별 파이싼 환경 변수가 다르다**. 일반 사용자는 /home/username/.local/bin 등이 포함된 PATH를 우선 사용하고 root 계정은 /usr/bin 또는 /usr/local/bin이 우선 사용한다. 아래 명령어로 파이썬 환경 변수를 확인할 수 있다.

```bash
# 일반 사용자
which python
python --version

# sudo로 실행
sudo which python
sudo python --version
```
