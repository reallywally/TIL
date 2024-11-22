# 개요

- 파이썬에서 많이 사용하는 lint를 비교해보고 우리 상황에 가장 적합한 친구를 찾아보자

## 비교 대상

### pylint

- 속도가 다른 lint 대비 했을때 느리다는 평이 많고 VSCode 후기에도 외국인들이 테러블이라는 단어를 사용할 정도
- VSCode 다운로드수는 200만이 조금 넘고, 깃헙[https://github.com/pylint-dev/pylint] 스타는 5.3k
- 적용 방법
  - pip install pylint로 설치
  - VSCode 익스텐션에서 pylint 검색해서 설치
- 간단 후기
  - doc 작성과 같이 너무 검사를 깐깐하게 해서 불편
  - .pylintrc 작성으로 검사를 disable할 수 있으나 이럴꺼면 pylint를 사용하는 의미가 없는거 같음

### flake8

- VSCode 다운로드수는 110만이 조금 넘고, 깃헙[https://github.com/PyCQA/flake8] 스타는 3.5k
- 적용 방법
  - pip install flake8 설치
  - VSCode 익스텐션에서 flake8 검색해서 설치
- 간단 후기
  - 특이사항 없음
  
### black

- VSCode 다운로드수는 390만이 조금 넘고, 깃헙[https://github.com/psf/black/tree/main] 스타는 39.2k
- 적용 방법
  - pip install black로 설치
  - VSCode 익스텐션에서 Black Formatter 검색해서 설치
- 간단 후기
  - 특이사항 없음

## 정리

### 결론

- pylint는 너무 깐깐해서 불편해서 패스
- flake8랑 black의 차이점은 체감할 수 없었으나 다운로드 수와 깃헙 스타수가 많은 **black**이 좋을것으로 생각

### 기타

- lint 설정하면 인텔리제이는 pep8에 맞지 안는 부분을 표시해주는데 VSCode는 저장할때 자동 포맷팅해서 신경을 안써도됨
  - 신경 안쓰는게 장점이기도 하지만 너무 pep8을 모르게 되는건 단점 같은
- 인텔리제이/파이참에서 black을 쓸려면 툴 버전이 2023.2 이후껏만 가능

### 참고자료

- <https://f-lab.kr/blog/flake8-and-pylint-and-black>
- <https://black.readthedocs.io/en/stable/integrations/editors.html>
