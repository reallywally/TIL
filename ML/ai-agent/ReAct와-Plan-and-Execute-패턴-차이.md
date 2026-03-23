# ReAct와 Plan and Execute 패턴 차이

## AI Agent 유형(패턴)을 찾다보면

ReAct랑 Plan and Execute를 분리해서 설멸하는 글도 있고 묶어서 설명하는 경우도 있다. 개념적인 내용이다 보니 뭐가 맞는지 혼동되어 내용을 정리해 본다.

쉽게 설명하면 ReAct와 Plan & Execute는 추상화 레벨이 다르다. ReAct는 LLM이 추론하는 마이크로 패턴이고, Plan & Execute는 작업을 어떻게 분해하고 실행하는 매크로 아키텍처이다. 그래서 Plan & Execute 안에 ReAct가 들어갈 수 있다.

각각의 예시를 좀 더 자세히 보면 다음과 같다.

### ReAct - LLM이 한 스텝씩 즉흥적으로 판단

``` text
질문 들어옴
→ Thought: 뭘 해야 하지?
→ Action: 툴 실행
→ Observation: 결과 보고
→ Thought: 다음엔 뭘 하지?
→ ... 반복
→ Final Answer
```

### Plan & Execute - 먼저 전체 계획 세우고, 순서대로 실행

```text
질문 들어옴
→ [Planner] 전체 계획 수립
   1. 사용자 취향 조회
   2. 장르별 최신작 검색
   3. 결과 종합해서 추천
→ [Executor] 1번 실행
→ [Executor] 2번 실행
→ [Executor] 3번 실행
→ Final Answer
```

그런데 왜 같이 묶어 설명하는 글이 있냐면 Plan & Execute의 Executor 부분이 내부적으로 ReAct로 동작할 수 있기 때문이다.

### ReAct를 포함한 Plan & Execute 아키텍처

```text
Planner (LLM)
  → "1. 취향 조회, 2. 검색, 3. 추천" 계획 수립

Executor1 (취향 조회, ReAct 에이전트)
  계획의 각 스텝을 ReAct 루프로 실행
  → Thought: 뭘 해야 하지?
  → Action: 툴 실행
  → Observation: 결과 보고
  → Thought: 다음엔 뭘 하지?
  → ... 반복
  → Final Answer
Excutuor2 (검색, ReAct 에이전트)
 → Executor1와 동일한 루프 실행
```

그래서 "ReAct는 Plan & Execute의 부품이다" 라고 볼 수도 있고, "둘은 독립적인 패턴이다" 라고 볼 수도 있다.

## 참고자료

- 분리해서 설명: <https://wikidocs.net/333764>
- 통합해서 설명: <https://yozm.wishket.com/magazine/detail/3078/>
