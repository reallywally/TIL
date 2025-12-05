# CrewAI 핵심개념

## Agent, Task, Crew

CrewAI는 Agent, Task, Crew 3가지 핵심 개념으로 구성된다. 그리고 각각의 개념을 고객센터팀을 예시로 설명하면 다음과 같다.

### 1. Agent

Agent는 특정 기술과 목표를 가진 팀원이다. 고객센터팀에는 앞단에서 고객을 응대하는 문의 담당자, 그리고 실제로 환불을 처리해주는 환불 처리 담당자가 있을 것이다. 그리고 CrewAI는 단순히 팀원(Agent)에 일만 시키는게 아니라 역할(Role), 목표(Goal), 배경 이야기(Backstory)를 가진 페르소나이다.

- Agent 1: 문의 담당자 (Triage Agent)
  - Role: 수석 고객 지원 분류가
  - Goal: 고객의 메시지를 분석하여 '환불', '기술 지원', '일반 문의' 중 하나로 정확히 분류한다.
  - Backstory: 당신은 수천 건의 티켓을 처리한 베테랑으로, 고객의 숨겨진 의도까지 파악합니다.

- Agent 2: 환불 처리 담당자 (Refund Agent)
  - Role: 환불 정책 관리자
  - Goal: 회사의 규정에 따라 해당 고객이 환불 대상인지 판단하고 승인 여부를 결정한다.
  - Backstory: 당신은 꼼꼼하고 원칙을 준수하지만, 공정한 판단을 내리는 전문가입니다.

### 2. Task

Task는 Agent가 수행하는 구체적인 작업이다. 방금 언급한 Agent를 보면 문의 담당자는 문의 종류가 무엇인지 정확하게 판단하고, 환불 처리 담당자는 실제로 환불이 가능한 상태인지를 판단하고 환불을 진행해야한다. Task 또한 Agent와 마찬가지로 페르소나를 가지며 설명(Description), 예상 결과(Expected Output), 담당자(Agent)로 구성된다.

- Task 1: 분류 작업
  - Description: 고객의 입력 텍스트("{customer_input}")를 분석하고 카테고리를 지정하세요.
  - Expected Output: 'Refund', 'Tech_Support', 'General' 중 하나의 단어만 반환.
  - Agent: 문의 담당자

- Task 2: 환불 심사 작업
  - Description: 앞선 분류 결과가 'Refund'인 경우, 고객의 구매 일자를 확인(가상 도구 사용)하고 7일 이내인지 판단하여 환불 승인 여부를 결정하세요.
  - Expected Output: 환불 승인 여부(Yes/No)와 그 이유가 담긴 짧은 보고서.
  - Agent: 환불 처리 담당자

### 3. Crew

Crew는 Agent와 Task를 하나로 묶은 팀이다. 고객지원팀이 어떤 업무를 어떻게 할지 결정하고 실행한다. Crew의 페르소나는 다음과 같다.

- Process: Sequential (순차적 방식)
  - 사용자 입력 → Agent 1이 분류 → 그 결과를 받아 Agent 2가 심사 → 최종 답변 생성
- Input: 고객이 챗봇 창에 입력한 질문
- Output: 챗봇이 사용자에게 보여줄 최종 답변

## 코드로 보는 구조

## 궁금했던 내용

### 1개의 Agent는 1개의 Task만 할수 있나?
