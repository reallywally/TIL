# pip-audit을 알아보자

## 무엇인고

파이썬 프로젝트의 의존성 패키지에 알려진 보안 취약점(CVE)이 있는지 확인하고 업그레이드 경로를 제시해 주는 도구이다.

## 사용방법

pip으로 설치하고 명령어만 입력하면 취약점을 검사하고 보안 취약점에 걸린 ID랑 수정해야할 버전까지 결과를 직관적으로 보여준다.

```shell
# 설치
pip install pip-audit

# 활성화된 가상환경으로 보안 취약점 검사
pip-audit 

# 결괴
Name              Version ID                  Fix Versions
----------------- ------- ------------------- ------------
cryptography      46.0.5  PYSEC-2026-35       46.0.6
cryptography      46.0.5  PYSEC-2026-36       46.0.7
cryptography      46.0.5  PYSEC-2026-36       46.0.7
cryptography      46.0.5  PYSEC-2026-35       46.0.6
cryptography      46.0.5  GHSA-537c-gmf6-5ccf 48.0.1
pip               25.0.1  PYSEC-2026-196      26.1.2
pip               25.0.1  PYSEC-2026-1795     25.3
pip               25.0.1  PYSEC-2026-1796     26.0
pip               25.0.1  PYSEC-2026-196      26.1.2
pip               25.0.1  PYSEC-2026-2875     26.1
pip               25.0.1  PYSEC-2026-2876     26.1
pydantic-settings 2.13.1  GHSA-4xgf-cpjx-pc3j 2.14.2
pyjwt             2.11.0  PYSEC-2026-120      2.12.0
pyjwt             2.11.0  PYSEC-2026-120      2.12.0
pyjwt             2.11.0  PYSEC-2026-179      2.13.0
pyjwt             2.11.0  PYSEC-2026-175      2.13.0
pyjwt             2.11.0  PYSEC-2026-177      2.13.0
pyjwt             2.11.0  PYSEC-2026-178      2.13.0
pyjwt             2.11.0  PYSEC-2026-176      2.12.1
pyjwt             2.11.0  PYSEC-2026-177      2.13.0
pyjwt             2.11.0  PYSEC-2026-179      2.13.0
pyjwt             2.11.0  PYSEC-2026-176      2.13.0
pyjwt             2.11.0  PYSEC-2026-178      2.13.0
starlette         0.46.2  PYSEC-2026-161      1.0.1
starlette         0.46.2  PYSEC-2026-161      1.0.1
starlette         0.46.2  PYSEC-2026-248      1.3.0
starlette         0.46.2  PYSEC-2026-249      1.3.1
starlette         0.46.2  PYSEC-2026-248      1.3.0
starlette         0.46.2  PYSEC-2026-1942     0.49.1
starlette         0.46.2  PYSEC-2026-1941     0.47.2
starlette         0.46.2  PYSEC-2026-2281     1.1.0
starlette         0.46.2  PYSEC-2026-2280     1.1.0

```

## 정리

보안은 중요하니 파이썬 패키지를 추가하면 검사해보자.
