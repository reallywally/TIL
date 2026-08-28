# Claude Code의 agents · skills · plugins를 알아보자

계기: OpenExecutive 저장소의 `.claude/` 구조를 분석하다가, 내가 예전에 설치한 `ponytail` 플러그인은 언제 실행되는지가 궁금해져서 셋을 비교하게 됨.

## 한 줄 요약

셋은 **경쟁 관계가 아니라 계층 관계**다.

> **agents**(일꾼) 와 **skills**(절차서) 는 *구성 요소*이고,
> **plugin**은 그것들 + **hooks**를 묶어 배포하는 *포장 단위*다.

그리고 이들을 가르는 진짜 기준은 "무엇을 하느냐"가 아니라 **누가 실행 시점을 결정하느냐**다.

| | 실행 결정 주체 | 성격 |
| --- | --- | --- |
| `.claude/agents/` | **모델의 판단** | 확률적 |
| `.claude/skills/` | **모델의 판단** 또는 사용자의 `/명령` | 반결정적 |
| plugin의 **hooks** | **하네스(런타임)의 이벤트** | 결정적 · 강제 |

## 1. `.claude/agents/` — 서브에이전트 정의

**정체**: 별도 세션으로 띄울 "일꾼"의 명세서. 마크다운 파일 하나 = 에이전트 하나.

```markdown
---
name: anvil-security-reviewer
description: >
  Hostile security reviewer. Use after code changes to adversarially audit
  the staged git diff for injection, auth bypass, hardcoded secrets...
tools: Read, Grep, Glob, Bash
model: opus
color: red
---

You are a hostile security reviewer. Assume the code is vulnerable until proven otherwise.
(...본문이 그 에이전트의 시스템 프롬프트가 된다)
```

### 로딩 방식 ★

세션 시작 시 **frontmatter만**(name + description) 메인 컨텍스트에 목록으로 올라간다. **본문은 안 올라간다.**
→ 호출되는 순간에야 본문이 그 서브에이전트의 시스템 프롬프트로 주입된다.

### 핵심 특성

- **컨텍스트가 격리된다.** 서브에이전트는 부모의 대화 맥락을 **못 본다.** 넘겨받은 프롬프트만 본다.
  - 장점: 그쪽에서 파일을 아무리 뒤져도 부모 컨텍스트가 안 더러워진다. **결론만 회수**한다.
  - 한계: **프롬프트 한 줄로 자기 일감을 특정할 수 있는 작업**만 떼어낼 수 있다.
- **모델을 개별 지정**할 수 있다 (opus/sonnet/haiku). 비용·깊이를 역할별로 다르게 가져감.
- **툴 권한을 제한**할 수 있다 (`tools:` 필드).
- 호출은 **모델 재량** → 안 부를 수도, 엉뚱한 걸 부를 수도 있다.

## 2. `.claude/skills/` — 절차·지식 묶음

**정체**: "이런 상황에서는 이렇게 해라"를 적어둔 절차서. `SKILL.md` 파일.

```markdown
---
name: ponytail
description: >
  Forces the laziest solution that actually works... Use on ANY coding task.
---
(...절차 본문)
```

### agents와 결정적으로 다른 점 ★

**스킬은 새 세션을 안 띄운다. 내(메인 에이전트) 컨텍스트에 본문이 로드되고, 내가 직접 그대로 수행한다.**

| | agents | skills |
| --- | --- | --- |
| 실행 주체 | 별도 세션의 서브에이전트 | 메인 에이전트 **본인** |
| 컨텍스트 | 격리 (부모 맥락 못 봄) | 공유 (현재 대화 그대로) |
| 결과 | 최종 보고서만 회수 | 작업 과정 전체가 그대로 보임 |
| 비용 | 추론 세션 추가 발생 | 컨텍스트 토큰만 증가 |

### 호출 경로 2가지

1. **사용자가 `/skill-name`** 으로 명시 호출
2. **모델이 description을 보고** 필요하다 판단해 호출

## 3. plugin — 배포 단위 (+ hooks라는 결정적 무기)

**정체**: 위의 것들을 **한 덩어리로 묶어 배포**하는 패키지. `ponytail` 실제 구조:

```
ponytail/
├── .claude-plugin/
│   ├── plugin.json        ← 매니페스트 (name, version, hooks 경로)
│   └── marketplace.json   ← 마켓플레이스 등록 정보
├── commands/              ← 슬래시 명령 6개 (*.toml)
├── skills/                ← 스킬 6개 (각 SKILL.md)
├── hooks/                 ← ★ 여기가 핵심
│   ├── claude-codex-hooks.json  ← 이벤트 ↔ 스크립트 매핑
│   └── *.js                     ← 실제 훅 스크립트
└── ponytail-mcp/          ← MCP 서버까지 동봉 가능
```

### hooks — 유일하게 "강제"되는 메커니즘 ★★

agents/skills가 **모델의 판단**에 의존하는 반면, **hooks는 하네스가 정해진 이벤트에 무조건 실행한다.** 내 판단이 개입할 여지가 없다.

`ponytail`의 실제 훅 등록:

| 이벤트 | 발화 시점 | 하는 일 |
| --- | --- | --- |
| `SessionStart` (`startup\|resume\|clear\|compact`) | 세션 시작·재개·`/clear`·컨텍스트 압축 직후 | 룰셋을 **숨은 컨텍스트로 주입** |
| `SubagentStart` | 서브에이전트가 뜰 때마다 | 서브에이전트에도 같은 룰 주입 |
| `UserPromptSubmit` | 프롬프트 보낼 때마다 | `/ponytail` 명령 감지해 모드 전환 |

- 각 훅에 `timeout` (ponytail은 5초) 설정.
- `compact` 매처가 있는 이유: **컨텍스트 압축 때 주입한 룰이 날아가므로 재주입**해야 함. → 훅 설계 시 반드시 고려할 포인트.
- 이 밖에 `PreToolUse` / `PostToolUse` / `Stop` / `PermissionRequest` 등 다수의 이벤트 존재.

> **"항상 반드시 X 해라"** 는 요구는 memory나 프롬프트로는 보장이 안 된다. **hooks로만 강제된다.**

## 전체 비교표

| 구분 | `.claude/agents/` | `.claude/skills/` | plugin |
| --- | --- | --- | --- |
| 단위 | 서브에이전트 1개 | 절차서 1개 | **묶음 배포** |
| 실행 주체 | 별도 세션 | 메인 에이전트 | (내부 구성물에 따라) |
| 컨텍스트 | **격리** | 공유 | — |
| 트리거 | 모델 판단 | 모델 판단 / `/명령` | **훅 = 이벤트 강제** |
| 모델 지정 | ✅ 가능 | ❌ | — |
| 상시 로드 비용 | description만 (저렴) | description만 (저렴) | 훅은 매 이벤트 실행 |
| 설치/공유 | 파일 복사 | 파일 복사 | **마켓플레이스 · 버전 관리 · 자동 업데이트** |

## 실제 사례 2개 — 정반대 철학

### A. OpenExecutive (SenteLabsAI) — agents + skills 조합

**사후(事後) 적대적 검증** 패턴.

- `.claude/agents/` 에 리뷰어 3명:

  | 에이전트 | 모델 | 역할 |
  | --- | --- | --- |
  | `anvil-security-reviewer` | opus | 인젝션·인증우회·시크릿·race condition |
  | `anvil-logic-reviewer` | sonnet | off-by-one·엣지케이스·상태전이 |
  | `anvil-quality-reviewer` | haiku | 40줄 초과 함수·매직넘버·중복 |

- **모델 티어를 일부러 분산**(deep/balanced/fast)해 관점 다양성 확보.
- 프롬프트가 전부 `"Assume the code is wrong until proven otherwise"` — **유죄추정**. 동시에 `"Do not invent issues"`로 환각 제동.
- 출력 계약: 마지막 줄에 반드시 `VERDICT: PASS` 또는 `VERDICT: FAIL` → **기계 판독 가능**.
- 입력은 항상 `git --no-pager diff --staged` 를 **에이전트가 직접 읽는다** (부모가 diff를 넘기지 않음 = 컨텍스트 절약).

**배운 점 ★**: 이 저장소는 **모델의 호출 판단을 신뢰하지 않는다.** `anvil` 스킬이 결정론적으로 지휘한다.

- 작업 크기별 규칙: Small=리뷰 없음 / Medium=보안 1명 / Large·위험파일=**3명 병렬**
- 위험(🔴) 파일 = 인증·암호화·결제, 데이터 삭제, 스키마 마이그레이션, 동시성, 공개 API → 한 줄 수정도 Large로 승격
- **SQLite 원장에 리뷰 레코드를 INSERT하고, 개수가 안 맞으면 다음 단계 진행 차단** (게이트)
- 재시도는 최대 2라운드, 이후엔 미해결 이슈를 명시하고 `Confidence: Low`로 보고

### B. ponytail — plugin + hooks

**사전(事前) 행동 제약** 패턴. 방향이 정반대다.

- 정체: *"Lazy senior dev mode"* — YAGNI, stdlib 우선, 요청 안 한 추상화 금지.
- SessionStart 훅으로 **세션 열리는 순간 룰셋을 강제 주입**. 내가 부르는 게 아니라 **이미 들어와 있는 상태로 시작**.
- 모드 5단계: `off / lite / full / ultra / review`, 기본 `full`.
  - `/ponytail lite` → 세션 한정
  - `/ponytail default lite` → 설정 파일에 영구 저장
  - 환경변수 `PONYTAIL_DEFAULT_MODE` 도 지원

> **대비**: anvil = 만든 뒤 두들겨 팬다 / ponytail = 애초에 크게 못 만들게 막는다.

## 스코프 — 어디에 두느냐가 적용 범위를 결정

| 위치 | 범위 |
| --- | --- |
| `<프로젝트>/.claude/agents,skills/` | 그 저장소에서만. **git에 커밋되어 팀 전체 공유** |
| `~/.claude/agents,skills/` | 내 모든 프로젝트 |
| `~/.claude/settings.json` → `enabledPlugins` | 전역 플러그인 활성화 |
| `<프로젝트>/.claude/settings.local.json` → `enabledPlugins` | **그 프로젝트에서만** 플러그인 활성화 |

### 실제로 겪은 함정 ★

`ponytail`이 설치돼 있는데도 안 돌아서 확인해보니, `~/.claude/plugins/installed_plugins.json` 에 **scope: "local"** 로 특정 프로젝트 2곳에만 묶여 있었다.

- 마켓플레이스는 **전역 등록**(`extraKnownMarketplaces`) 되어 있어도
- 플러그인 **활성화는 프로젝트별**(`settings.local.json`)일 수 있다

→ **"설치했다" ≠ "지금 여기서 돈다".** 헷갈리면 확인할 곳:

```
~/.claude/plugins/installed_plugins.json     # 어디에 설치됐나
<프로젝트>/.claude/settings.local.json        # 여기서 켜졌나
~/.claude/.ponytail-active                    # (플러그인별) 활성 플래그
```

## 선택 기준 — 언제 뭘 쓸까

| 상황 | 선택 |
| --- | --- |
| 컨텍스트를 더럽히지 않고 **탐색·검증**을 시키고 싶다 | **agent** |
| 역할별로 **다른 모델**을 쓰고 싶다 (비용 최적화) | **agent** |
| 여러 관점으로 **병렬 검토**가 필요하다 | **agent** ×N |
| 반복되는 **절차·체크리스트**를 고정하고 싶다 | **skill** |
| 진행 과정을 **내가 계속 보면서** 가야 한다 | **skill** |
| **예외 없이 항상** 적용돼야 한다 | **hooks (plugin)** |
| 남에게 **배포·버전 관리**하고 싶다 | **plugin** |

## 함정 정리

1. **"정의 개수"는 싸고 "호출 횟수"는 비싸다.**
   - 정의: description 한두 줄만 상시 로드 → 10개든 30개든 무시할 만함
   - 호출: 서브에이전트 1개 = **독립된 풀 추론 세션**. Large 리뷰 3명 병렬 × 최대 2라운드 = 추론 6회

2. **진짜 병목은 라우팅 품질이다.**
   에이전트가 늘수록 description이 겹치고 → 모델의 선택 정확도가 떨어진다. **툴을 너무 많이 붙였을 때와 동일한 실패 양상.** OpenExecutive가 3개로 끊고 보안/로직/품질을 **겹치지 않게** 자른 이유.

3. **프롬프트 중복은 부채가 된다.**
   ponytail 캐싱 감시 조항이 security·quality **두 파일에 중복**돼 있다. 규칙이 바뀌면 둘 다 고쳐야 하는데 강제 장치가 없다.

4. **서브에이전트는 대화 맥락을 못 본다.** 설계 시 가장 먼저 걸리는 제약.

5. 대략적 기준: 정의 **5~10개면 무난**, **20개 넘어가면** description 중복 점검 필요.

## 덤 — 프롬프트 캐싱을 자동 게이트로 만든 사례

OpenExecutive의 security·quality 리뷰어 프롬프트에는 **저장소 전용 조항**이 박혀 있다. diff가 `prompts/cache_manager.py`, `prompts/executive_persona.py`, `memory/company_profile.py`, 또는 아무 `cache_control` 블록을 건드리면:

- `cache_control` 붙은 시스템 블록에 **동적 콘텐츠**(f-string, `.format()`, 문자열 연결, RAG 컨텍스트)가 들어갔는가
- 페르소나가 상수가 아니라 **f-string으로 조립**됐는가
- **툴 정의가 이름순 정렬**이 안 됐는가

를 검사한다. 그리고 이걸 **HIGH 심각도**로 취급하라고 지시한다. 이유가 인상적:

> *"it silently ~10x's API cost"*

**보안 취약점이 아니라 비용 사고를 보안 등급으로 격상**시킨 것. 사람 리뷰에 맡기지 않고 자동 게이트로 만들었다.

## 참고

- OpenExecutive: <https://github.com/SenteLabsAI/OpenExecutive>
- ponytail: <https://github.com/DietrichGebert/ponytail>
