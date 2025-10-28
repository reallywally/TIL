# Is Subsequence 복기

## 내가 만든 코드

```python
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        s_index = 0
        
        for char in t:
            if s_index < len(s) and char == s[s_index]:
                s_index += 1

        return s_index == len(s)
```

## 베스트 정답

```python
class Solution:
    def isSubsequence(s, t):
        i = 0
        j = 0
        
        while i < len(s) and j < len(t):
            if s[i] == t[j]:
                i += 1
            j += 1
        
        return i == len(s)
```

## 핵심 포인트

- ❌ 틀린 생각: s의 각 문자를 t에서 찾으면 되지 않나?
- ✅ 올바른 생각: t를 한 방향으로만 읽으면서 s의 문자를 순서대로 찾기
- 시간복잡도 O(n)으로 문제 해결

## 이 패턴을 실무에서 마주친다면

이런 알고리즘 문제는 풀때는 풀 수 있는데 이런 패턴을 실제 환경에서는 언제 활용하는기 알기 어렵다.
클로드씨에게 물어보니 아래와 같은 경우라면 잘 활용할 수 있을거 같다.

### 1. 사용자 행동 추적

```python
# 특정 순서의 이벤트가 발생했는지 확인
expected_flow = ["login", "add_to_cart", "checkout", "payment"]
user_events = ["login", "browse", "add_to_cart", "browse", "checkout", "payment", "logout"]

# 정상적인 구매 플로우를 거쳤는가?
def check_conversion_funnel(expected, actual_events):
    i = 0
    for event in actual_events:
        if i < len(expected) and event == expected[i]:
            i += 1
    return i == len(expected)
```

### 2. 비정상 접근 패턴 감지

```python
# 해커가 특정 순서로 공격하는 패턴
suspicious_pattern = ["port_scan", "login_attempt", "privilege_escalation"]
user_actions = ["browse", "port_scan", "api_call", "login_attempt", "privilege_escalation"]

def detect_attack_pattern(pattern, actions):
    i = 0
    for action in actions:
        if i < len(pattern) and action == pattern[i]:
            i += 1
            if i == len(pattern):
                return True  # ⚠️ 경고!
    return False
```

## 후기

내 코드가 조건문으로 예외처리가 필요하지만 베스트 정답이랑 비교하면 나쁘진 않을거 것같다.
