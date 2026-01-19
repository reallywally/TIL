# 백엔드 개발자를 위한 LangGraph 에이전트 개발 가이드

## 들어가며

보통 백엔드는 Controller, Service, Repository 처럼 많이 사용하는 구조이 있다. 그런데 AI Agent는 이런 구조가 아니다. 그래서 막상 AI Agent를 개발하려 하니 구성을 어떻게 해야될지 잘 이해가 안됬다. 이번 글에서는 Controller, Service, Repository 처럼 AI Agent에서 일반적으로 사용하는 구조에 관하여 학습한 내용을 정리해본다.

## 1. Contoller, Service, Repository 구조 vs LangGraph 구조 비교

완벽하게 매칭되는건 아니지만 LangGraph를 기준으로 역할울 정리하면 다음과 같다.

| 구분 | 전통적인 웹 | LangGraph 에이전트 | 역할 설명 |
| --- | --- | --- | --- |
| 데이터 모델 | DTO | **State (상태)** | 전체 프로세스 동안 유지되고 업데이트되는 공통 데이터 보관소 |
| 비즈니스 로직 | Service | **Nodes (노드)** | LLM 호출, 데이터 가공, 도구 실행 등 실제 "작업"을 수행하는 단위 |
| 흐름 제어 | Controller | **Edges (엣지)** | 다음 작업으로 갈지, 루프를 돌지, 끝낼지 결정하는 "분기점" |
| 외부 데이터 | Repository | **Tools (도구)** | DB 조회, 검색 API 등 에이전트가 손발처럼 사용하는 외부 기능 |
| 오류 처리 | try-catch, 전역 핸들러 | **Retry 노드, Fallback Edge** | 실패 시 재시도 또는 대체 경로 실행 |

## 2. LangGraph의 핵심 역할 상세 설명

### 2.1 State

게시판에서 DTO가 한 번 전달되고 끝난다면, LangGraph의 State는 에이전트가 작업을 끝낼 때까지 계속 들고 다니는 "작업 노트"이다. react의 useState처럼 상태관리라 생각해도 될것 같다.

- 역할: 대화 내역, 검색된 결과물, 현재 단계 등을 저장
- 특징: 노드를 거칠 때마다 이 상태가 조금씩 업데이트(Overwrite 또는 Append)

### 2.2 Nodes

특정 기능 또는 비즈니스 로직을 수행하며 전통적인 Service 클래스의 메서드와 비슷하다.

- LLM 노드: "상태"를 보고 다음에 뭘 할지 추론
- 정제 노드: 결과물을 사용자에게 보여주기 좋게 다듬기

### 2.3 Edges

Controller가 URL에 따라 서비스를 호출하듯, Edge는 노드 간의 길을 연결한다.

- Normal Edge: 작업 A가 끝나면 무조건 B로 이동합니다.
- Conditional Edge (핵심): LLM의 판단 결과에 따라 "도구를 더 써야 할까(Loop)?" 아니면 "이제 답변을 줄까(End)?"를 결정합니다.

### 2.4 Tools

사실 Tools의 역할을 Repository보다 더 넓다. Repository는 DB 접근이지만, Tools는 외부 API 호출 (날씨, 인터넷 검색 등), 파일 시스템 접근, 계산/변환 작업, DB 조회 등이다.

### 3. 게시판을 비유로 에이전트 동작 프로세스 예시

"지난주에 올라온 게시글 요약해줘"라는 요청이 들어왔을 때의 흐름이다.

1. 시작: 사용자의 질문이 State에 저장
2. LLM 노드 (Service 역할): State를 보고 어떤 Tool이 필요할지 판단하여 "DB 조회 Tool" 사용
3. Conditional Edge (Controller 역할): 게시글이 있으면 요약 LLM 노드로 보내고, 없으면 End
4. LLM 노드 (Service 역할): 추가된 데이터를 보고 요약문을 작성 후 End

### 4. 왜 이렇게 개발할까?

전통적인 방식은 모든 비즈니스 로직을, 예외 케이스를 개발자가 직접 코딩해야 하지만, 에이전트 구조는 **"판단은 LLM(노드)에게 맡기고, 개발자는 큰 흐름(그래프 구조)과 가드레일만 설계"**하기 위해서이다.
