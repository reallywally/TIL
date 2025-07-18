# cursor를 써보자

25년 5월 23일 기준 개발툴로 프론트는 vscode, 백엔드는 인텔리제이를 사용하고 있다. 그리고 든든한? 친구 코파일럿으로 개발을하고 있는데 생각보다 생산성이 안나오고 있었다. 코파일럿도 처음에는 좋았는데 막상 쓰다보면 잘못 추론하는 경우도 생기고 chatGPT, 클로드도 사용하지만 AI 사용을 위해 크롬과 툴을 자주 왔다갔다 해서 불편했다. 인텔리제이는 21년 버전을 폴백 라이센스로 쓰고 있어 MCP 같은 최신 기능이 안되기도하고 내돈 주고 사기는 좀 아까웠다.  
그래서 생산성 향상을 위해 여러 방법을 찾다가 가장 돈이 적게 들면서 좋다고 유명한 방법을 찾다보니 cursor가 좋아보였다. 그리고 그 과정에서 필요했던 내용이나 메모를 정리해 본다. cursor는 vscode를 포크해서 만들거기 때문에 일부 vscode 관련 내용도 포함하였다.

## 정리

### vscode로 스프링 개발이 가능 할까?

- sout, iter 등 인텔리제이에서 자주 쓰는 숏컷이 지원함

### cursor에서 vscode 설정 가져오기

- 메뉴에서 File > Preference > Cursor Settings 또든 윈도우 오른쪽 설정 버튼을 클릭하여 설정 메뉴로 이동
- VS Code Import에서 Import 버튼을 클릭
- VS Code는 계정으로 확장 프로그램과 설정이 동기화 되는데 cursor는 안되는것 같음

### 기타 팁, 확장 프로그램

- Remote - SSH

  - 서버에 접속할수 있는 확장 프로그램
  - cursor로 서버 파일을 열어 편집이 가능하고 터미널로 명령어 실행도 가능
      -> 서버에서 vim으로 편집하는것 보다 훨씬 편함
      -> 인텔리제이에도 있을법한 기능이긴한데 사용해본적이 없음
  - mobaXterm 대체로 사용가능해 보임
  - 조금 느리지만 불편한 정도는 아님

### 사용 소감

- 인텔리제이와 비교하면 여러 부분에서 부족한 부분이 느껴진다.
  - cursor의 속도가 느린거 같다. 특히 스프링 시작할때 차이가 심하다.
  - 파이썬 자동완성이 부족한거 같다.
  - git history보는데 UI가 불편하다.
  - DB접속 툴을 따로 써야한다.
- 디버그 모드는 처음엔 불편했는데 사용하다 보니 적응이 되었다.
- 툴 안에서 바로 물어보고 수정하니 GPT랑 클로드를 쓰는것 보단 편했다.
- 프롬프트는 잘 쓰면 좋겠으나 아직은 활용도가 낮다.
- 2025년 7월 18일 기준 제미나이는 무료로 많이 사용할 수 있고, 코딩 결과만 보면 클로드 코드가 좋다는 평가가 많다.
  - 위에서 언급했던 자동완성, git, 코드 검사 등 부분에서 적응하다가도 인텔리제이가 좋은건 어쩔 수 없다.
  - 지금 기준으로는 인텔리제이 + 클로드 코드가 가장 많이 사용하는 스펙 같다.
  