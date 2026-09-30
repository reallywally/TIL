# Orca 모노레포에서 worktree를 어떤 단위로 나눌까

최근 Orca로 바이브 코딩을 하고 있는데 워크트리라는 단위가 보였다. 뭔가 작업 단위로 분리하는거 같은데 학습한 내용을 정리해 본다. 내 프로젝트 구조는 이렇다.

```text
repo/
├── web/     # 프론트엔드
└── server/  # 백엔드
```

처음엔 자연스럽게 `web`용 워크트리, `server`용 워크트리로 나누려 했는데, 이게 알맞게 분리한건지 궁금했다.

## worktree란 무엇인가

사실 worktree는 orca에서 사용하는 개념이 아니라 **git**에서 사용하는 개념이다. 보통 브랜치를 옮길 땐 `git checkout`으로 **같은 디렉터리 안에서** 파일을 갈아끼운다.
그래서 브랜치 A 작업 중에 브랜치 B를 보려면 stash하거나 commit한 후에 원하는 브랜치로 checkout을 해야한다. 두 브랜치를 동시에 실행할 수는 없다.  
그래서 worktree라는 개념이 나왔다. `git worktree`는 **하나의 저장소에서 여러 브랜치를 각각 별도 디렉터리로 체크아웃**하는 기능이다.

```bash
git worktree add ../repo-feat-a feat/a   # ../repo-feat-a 에 feat/a 브랜치 체크아웃
git worktree add ../repo-fix-b fix/b
git worktree list                        # 현재 워크트리 목록
git worktree remove ../repo-feat-a       # 작업 끝나면 정리
```

```text
repo/            ← main
repo-feat-a/     ← feat/a
repo-fix-b/      ← fix/b
```

Orca가 하는 일은 이 git worktree를 시각화하여 진행 상태를 한눈에 보고 클릭하면서 이동할 수 있는것이다.

## 결론부터

> **1 워크트리 = 1 PR = 혼자 머지해도 앱이 깨지지 않는 기능 단위**

워크트리는 브랜치와 1:1로 묶이는 일회용 작업 공간이다. 그러니 분리 기준은 "이 변경을 PR 하나로 올릴 수 있는가?"로 잡으면 된다.

## 기준 1. 레이어가 아니라 기능으로 세로로 자른다

`web/`과 `server/`로 나누면 API 스펙 하나 바꿀 때마다 두 워크트리가 동시에 움직여야 한다.
한쪽만 머지하면 앱이 깨지고, 병렬 작업의 이점이 사라진다.

대신 **프론트 + 백엔드를 함께 건드리는 기능 하나**를 한 워크트리에 둔다.

```text
feat/order-history
├── server/  → 주문 내역 엔드포인트
└── web/     → 주문 내역 화면
```

에이전트 한 세션이 API와 UI를 같이 짜니 스펙 불일치도 안 생긴다.

## 기준 2. 한쪽만 바뀌는 작업은 진짜 독립적이다

이런 작업들은 파일이 거의 안 겹쳐서 병렬로 돌리기 가장 좋다.

- `fix/web-header-overflow` — web만
- `refactor/server-db-session` — server만
- `chore/web-eslint-rules` — web만

동시에 3~4개 띄워도 충돌이 거의 없다.

## 기준 3. 공통 계층 변경은 먼저, 그리고 혼자

아래 파일들은 여러 워크트리가 동시에 손대면 반드시 충돌한다.

- 루트 `package.json` / lock 파일, `pyproject.toml`
- 공유 타입·스키마 (Pydantic 모델 → 프론트 타입 생성 산출물 등)
- 라우터 등록 파일
- DB 마이그레이션

이런 변경은 **단독 워크트리로 먼저 머지**하고, 나머지 작업을 그 위에서 시작한다.

특히 Alembic 리비전 분기는 모노레포 병렬 작업에서 가장 자주 터지는 문제라,
마이그레이션이 필요한 작업은 동시에 하나만 돌린다.

## 기준 4. 에이전트 한 세션에 끝나는 크기

- 너무 크면 → 컨텍스트가 넘쳐 에이전트 품질이 떨어진다
- 너무 작으면 → 워크트리마다 `npm install`, dev 서버 띄우는 오버헤드가 더 크다

대략 반나절~하루 분량이 적당했다.
