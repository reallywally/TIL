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

앞서 설명한 내용을 다음과 같이 파이썬 코드로 정리할 수 있다.

```python
from crewai import Agent, Task, Crew, Process

# 1. Agents 정의 (직원 채용)
classifier = Agent(
    role='수석 분류 담당자',
    goal='고객 문의를 정확하게 카테고리화 한다',
    backstory='당신은 고객의 의도를 0.1초 만에 파악하는 분류의 신입니다.',
    verbose=True
)

refund_expert = Agent(
    role='환불 담당자',
    goal='규정에 의거하여 환불 가능 여부를 판단하고 응답을 작성한다',
    backstory='당신은 회사의 자산을 지키면서도 고객을 존중하는 전문가입니다.',
    verbose=True
)

# 2. Tasks 정의 (업무 지시서)
# 사용자의 입력은 {inquiry} 변수로 들어옵니다.
classify_task = Task(
    description='다음 고객 문의를 분석하세요: "{inquiry}"',
    expected_output='문의 유형 (환불/기술지원/기타)',
    agent=classifier
)

process_refund_task = Task(
    description='이전 분류 작업의 결과를 바탕으로, 환불 요청일 경우 규정을 검토하고 고객에게 보낼 정중한 답변을 작성하세요.',
    expected_output='고객에게 보낼 최종 답변 텍스트',
    agent=refund_expert,
    context=[classify_task] # 이전 작업의 결과를 참고함
)

# 3. Crew 정의 (팀 결성 및 실행)
support_crew = Crew(
    agents=[classifier, refund_expert],
    tasks=[classify_task, process_refund_task],
    process=Process.sequential, # 순서대로 작업 (1 -> 2)
    verbose=True
)

# 4. 챗봇 실행 (Kickoff)
user_input = "산 지 3일 됐는데 마음에 안 들어요. 돈 돌려주세요."
result = support_crew.kickoff(inputs={'inquiry': user_input})

print("챗봇 답변:", result)
```

## 궁금했던 내용

### 1개의 Agent는 1개의 Task만 할수 있나?

아니다. 명확한 업무를 위해 1:1로 매핑하는 경우가 많지만 반드시는 아니다. 1:N으로 예시를 들면 환불 담당자가 환불 완료 후 문자 보내는 업무까지할 수 있기 때문이다. 그리고 CrewAI의 가장 강력한 특징 중 하나로 언급되는 **위임(Delegation)** 이라는 기능이 있다는데 이건 CrewAI를 좀더 공부하게 되었을떄 정리해보자.
