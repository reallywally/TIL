# pycharm 무료버전으로 FastAPI 개발하자

젯브레인은 원래 Intellij와 Pycharm 등 개발 툴을 기능이 제한된 무료 버전의 커뮤니티와 많은 기능이 있는 유료 버전의 얼티메이트 버전이 있었다. 그런데 최근에는 제품은 1개로 통합하고 구독 여부에 따라서 기능이 제한되는걸로 변경되었다. VScode로 개발하는게 너무 불편한 와중에 Pycharm은 무료를 써도 크게 불편하지 않을것 같아 한번 써보기로 했다.

## FastAPI 실행하기

유로 버전은 간편하게 FastAPI 실행 설정이 가능한데 무료버전은 스크립트 실행을 설정해야 한다. 디버깅을 위해 Run Configuration으로 등록하는 방법을 추천한다.

- Run → Edit Configurations → + → Python 선택 후:
- module name으로 전환 (script path 말고): uvicorn
- Parameters: main:app --reload --port 8000
- Working directory: 프로젝트 루트

## ruff, pyright 설정

기본적으로 코드 정렬을 하면 Pycharm 자체 설정으로 한다. 요즘 많이 쓰는 ruff와 pyright를 설정 해보자.

- Settings (Ctrl+Alt+S) → Plugins → market place에서 ruff와 pyright를 설치
- Settings → Python → Tools → External Tools 에서 ruff와 pyright를 ON으로 설정
