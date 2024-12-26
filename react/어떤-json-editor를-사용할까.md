# 어떤 JSON Editor를 사용할까

## 상황

화면에서 JSON 객체를 편집하고 그대로 저장하는 기능이 필요했다. 여러 JSON Editor가 있는데 테스트해본 내용을 정리한다. 개발 시간 단축을 위해 커스텀 작업 없이 사용하기 편한 라이브러리를 선택하려 한다. 프론트 개발 환경은 React18, 타입스크립트이다.

## 후보군

1. react-json-view
    * 리액트 17까지만 지원해서 설치도 못함

2. react-json-editor-ajrm
    * 23년 1월 부터 아카이브되었음
    * git: <https://github.com/AndrewRedican/react-json-editor-ajrm>

3. jsoneditor
    * 타입스크립트랑 호환성이 안맞는거 같음
    * git: <https://github.com/josdejong/jsoneditor>

4. @uiw/react-json-view
    * 실행은 되나 JSON 문법 체크가 조금 느림
    * git: <https://github.com/uiwjs/react-json-view>

5. monaco-react
    * VS Code 처럼 편집해서 익숙하고 지금 개발 환경에 호환성이 잘 맞음
    * 자동 포맷팅이 있으면 좋을텐데 그건 안되는듯
    * git: <https://github.com/suren-atoyan/monaco-react?tab=readme-ov-file#usemonaco>
    * codesandbox: <https://codesandbox.io/p/sandbox/cdn-example-forked-s9v5r9?file=%2Findex.html%3A40%2C28>
