---
name: workmux-router
description: Natural-language task launcher and router using workmux and git worktree
---

# Skill: Workmux Router for Natural-Language Task Launching

## Purpose

이 스킬의 목적은 사용자가 Git 저장소 루트 또는 기존 worktree 내부에서 자연어로 작업을 지시했을 때,  
에이전트가 `workmux`를 백엔드 실행기로 사용하여 다음을 자동 수행하게 하는 것이다.

- 현재 작업을 기존 worktree에서 계속할지 판단
- 새 작업이면 새 worktree를 생성
- 적절한 세션을 열거나 새 세션을 시작
- 필요 시 작업 지시문을 해당 세션에 주입
- 사용자가 `workmux` 명령어를 직접 기억하지 않아도 되게 함

이 스킬은 **자연어 중심 작업 시작/재개 라우터**다.  
사용자는 `workmux add/open/dashboard` 같은 명령을 직접 호출하지 않는다.

---

## When to Use

다음 상황에서 이 스킬을 사용한다.

1. 사용자가 Git 저장소 루트에서 자연어로 새 작업을 지시할 때
2. 사용자가 기존 worktree 디렉토리 안에서 후속 작업을 지시할 때
3. 사용자가 PR 리뷰 반영, CI 실패 수정, 기능 개발, 조사, 문서 작업 등
   **작업 단위 분리**가 필요한 요청을 했을 때
4. 단일 저장소 내에서 여러 작업을 병렬로 분리해야 할 때
5. 작업 맥락 혼선을 줄이기 위해 기존 작업과 새 작업을 구분해야 할 때

---

## When NOT to Use

다음 경우에는 이 스킬을 사용하지 않는다.

1. 현재 디렉토리가 Git 저장소가 아니고, 저장소도 지정되지 않은 경우
2. 단순 조회성 작업으로 별도 작업 공간 분리가 필요 없는 경우
3. 사용자가 명시적으로 현재 worktree를 그대로 사용하라고 지시한 경우
4. 이미 자동화된 상위 런처가 동일한 판단을 수행하는 경우

---

## Core Principles

1. **한 worktree = 한 작업**
2. **한 worktree = 한 주요 브랜치/작업 컨텍스트**
3. **후속 작업은 가능하면 기존 worktree로 귀속**
4. **새로운 목적이면 새 worktree로 분리**
5. 사용자는 자연어로만 지시하고, workmux는 내부 구현으로 숨긴다
6. 불필요하게 worktree를 난립시키지 않는다
7. 현재 맥락을 유지할 수 있으면 기존 worktree를 우선 재사용한다

---

## Definitions

### Current Workspace
현재 에이전트가 실행 중인 디렉토리.  
Git 루트일 수도 있고, 특정 worktree 디렉토리일 수도 있다.

### Matching Task
현재 요청이 현재 worktree의 목적과 동일하거나 직접적인 후속 작업인 경우.

예:
- 같은 PR의 리뷰 반영
- 같은 브랜치의 CI 실패 수정
- 같은 기능 작업 이어서 진행
- 같은 버그의 후속 수정

### New Task
현재 worktree의 목적과 실질적으로 다른 별도 작업.

예:
- 다른 기능 개발
- 다른 버그 수정
- 다른 PR 대응
- unrelated 문서 작업
- unrelated 조사 작업

---

## Required Inputs

가능하면 다음 정보를 파악한다.

- 현재 디렉토리
- 현재 Git 루트
- 현재 브랜치
- 현재 디렉토리가 worktree인지 여부
- 기존 활성 worktree 목록
- 기존 worktree 이름/브랜치/작업 목적(가능한 경우)
- 사용자 자연어 요청

---

## High-Level Behavior

이 스킬은 아래 순서로 동작한다.

1. 현재 디렉토리가 Git 저장소인지 확인
2. 현재 디렉토리가 기존 worktree인지 판별
3. 사용자 요청이 현재 worktree의 목적과 일치하는지 판단
4. 일치하면 현재 worktree에서 계속 진행
5. 일치하지 않으면 기존 worktree 중 적합한 것이 있는지 탐색
6. 적합한 것이 있으면 해당 worktree/session으로 재개
7. 적합한 것이 없으면 새 worktree 생성
8. 새 또는 기존 worktree에서 세션을 시작하거나 재개
9. 필요 시 작업 지시문을 세션에 전달
10. 사용자에게 현재 연결된 작업 컨텍스트를 명확히 보여준다

---

## Decision Rules

### Rule 1. 현재 worktree 계속 사용
아래 중 하나면 현재 worktree를 계속 사용한다.

- 같은 기능/버그/리뷰 대응의 후속 작업
- 같은 PR에 대한 수정
- 같은 브랜치 목적과 일치
- "이어서", "계속", "추가로", "방금 작업한 거", "리뷰 반영", "CI 수정" 같은 후속 표현이 명확함

예:
- "이 코드리뷰 P1 반영해"
- "CI 깨진 거 수정해"
- "아까 작업 이어서 해"

### Rule 2. 기존 다른 worktree 재사용
현재 worktree와는 다르지만, 이미 존재하는 다른 worktree가 더 적합하면 그쪽을 연다.

예:
- 현재는 `feature/auth-refactor` 안에 있는데
- 요청은 "PR 482 리뷰 반영해"
- 이미 `review/pr-482-fixes` worktree가 있으면 그쪽으로 이동

### Rule 3. 새 worktree 생성
다음 경우 새 worktree를 만든다.

- 현재 작업과 unrelated한 새 기능/버그/조사
- 기존 worktree 중 적합한 것이 없음
- 명시적으로 새 작업으로 분리해야 하는 경우
- 긴급 수정으로 맥락 분리가 필요한 경우

예:
- "결제 timeout 버그 별도로 조사해"
- "주문 롤백 이슈 새로 봐줘"
- "이건 다른 작업이니 따로 진행해"

---

## Task Completion & Cleanup Rules

### Rule 1. 작업 완료 시 삭제 여부 확인
사용자가 "작업 완료", "끝났어", "머지했어", "정리해줘" 등 작업 종료를 알리는 의사를 표시하면:
1. 해당 워크트리에 저장되지 않은 변경사항이나 푸시되지 않은 커밋이 있는지 확인한다.
2. 사용자에게 **"작업이 완료되었습니다. 해당 워크트리와 세션을 삭제할까요?"**라고 명시적으로 질문한다.
3. 사용자가 승인(`Yes`)할 경우에만 `workmux remove` (또는 `git worktree remove`)를 실행하여 정리한다.
4. 사용자가 거절(`No`)할 경우 해당 워크트리를 보존하고, 나중에 `workmux open`으로 재개할 수 있음을 안내한다.

---

## Branch Naming Strategy

브랜치/작업 핸들은 사용자의 자연어 요청을 요약해 생성한다.

권장 패턴:

- `feature/<topic>`
- `fix/<topic>`
- `review/pr-<number>-fixes`
- `investigate/<topic>`
- `docs/<topic>`
- `chore/<topic>`

예:
- `feature/auth-refactor`
- `fix/ci-payment-timeout`
- `review/pr-482-p1-fixes`
- `investigate/order-rollback`
- `docs/restclient-migration`

### Naming Rules
- 짧고 의미 있게
- 작업 목적이 바로 드러나게
- temp, test, fix1 같은 모호한 이름 금지
- 가능하면 PR 번호나 핵심 토픽 포함

---

## Session Strategy

### 기본 원칙
- 새 worktree 생성 시 해당 worktree용 세션도 함께 생성 또는 재개한다
- 사용자는 가능한 한 직접 세션 전환 명령을 입력하지 않는다
- 자동 attach가 가능하면 attach를 우선한다
- 자동 attach가 어려우면 경로/세션명을 명확히 출력한다

### Preferred UX
1. 가능한 경우:
   - 새 worktree 생성
   - 새 세션 시작
   - 작업 지시문 주입
   - 해당 세션으로 사용자 포커스 이동

2. 불가능한 경우:
   - 새 worktree 경로 출력
   - 세션명 출력
   - 다음 진입 방법을 간단히 안내

---

## Natural-Language Routing Behavior

### Case A. Repo 루트에서 새 작업 시작
사용자가 repo 루트에서 자연어로 지시하면:

- 새 작업인지 판단
- 적절한 worktree가 없으면 새로 생성
- 세션 시작
- 지시문 전달

예:
- "인증 리팩터링 시작해"
- "PR 482 리뷰 반영해"
- "CI 실패 수정해"

### Case B. Existing worktree 안에서 작업 지시
사용자가 특정 worktree 안에서 지시하면:

- 현재 worktree와 요청의 관련성 판단
- 관련 있으면 그대로 진행
- 관련 없으면 새 worktree로 분리

예:
- 현재 `feature/auth-refactor` 안에서
  - "테스트 보강해" → 그대로 진행
  - "결제 timeout 버그 수정해" → 새 worktree 생성

---

## Minimal Execution Algorithm

### Step 1. Context detection
- 현재 경로 확인
- git root 확인
- 현재 브랜치 확인
- 현재 경로가 worktree인지 확인

### Step 2. Request classification
사용자 요청을 아래 중 하나로 분류한다.

- feature
- bugfix
- review-followup
- ci-fix
- investigation
- docs
- chore

### Step 3. Task match evaluation
현재 요청이
- 현재 worktree와 일치하는지
- 기존 다른 worktree와 일치하는지
판단한다.

### Step 4. Routing
- current-match → 현재 worktree 유지
- existing-match → 해당 worktree 열기
- no-match → 새 worktree 생성

### Step 5. Session handling
- 기존 세션이 있으면 재개
- 없으면 새 세션 시작
- 작업 프롬프트 전달

### Step 6. User feedback
사용자에게 다음을 명확히 보여준다.

- 선택된 worktree
- 브랜치명
- 새로 만들었는지 / 기존 것 재사용인지
- 현재 작업 목적
- 세션 연결 상태

---

## Output Style

작업 라우팅 후에는 아래 형식처럼 간단명료하게 보여준다.

### Reuse current worktree
- 현재 worktree를 계속 사용합니다.
- 작업 경로: `<path>`
- 브랜치: `<branch>`
- 사유: 현재 요청이 기존 작업의 후속 작업입니다.

### Reuse existing other worktree
- 기존 worktree를 재사용합니다.
- 작업 경로: `<path>`
- 브랜치: `<branch>`
- 사유: 해당 요청과 일치하는 기존 작업이 이미 존재합니다.

### Create new worktree
- 새 worktree를 생성합니다.
- 작업 경로: `<path>`
- 브랜치: `<branch>`
- 사유: 현재 요청은 기존 작업과 다른 별도 작업입니다.

---

## Safety Rules

1. 사용자의 확인 없이 main/master/release 브랜치에서 직접 작업하지 않는다
2. 기존 worktree를 삭제/머지하기 전에는 명시적 의도를 확인한다
3. 같은 요청을 여러 번 받아도 동일한 worktree가 있으면 중복 생성하지 않는다
4. 새 worktree를 만들기 전에 기존 유사 worktree 존재 여부를 먼저 확인한다
5. 후속 작업은 가능한 한 기존 작업 공간에 귀속한다
6. unrelated 작업은 반드시 분리한다

---

## Heuristics for Matching Current Worktree

아래 신호가 많을수록 현재 worktree 재사용 쪽으로 기운다.

- 같은 PR 번호 언급
- 같은 이슈/토픽 키워드
- "이어서", "계속", "리뷰 반영", "CI 수정" 표현
- 현재 브랜치명과 사용자 요청 주제의 높은 유사성

아래 신호가 많을수록 새 worktree 생성 쪽으로 기운다.

- 전혀 다른 도메인/기능 언급
- 다른 PR 번호 언급
- 다른 버그/기능명
- "이건 별도로", "새로", "따로", "분리해서" 표현

---

## Suggested Internal Commands

이 스킬은 내부적으로 다음 종류의 작업을 사용할 수 있다.

- 현재 git root 탐색
- 현재 브랜치 확인
- 기존 worktree 목록 조회
- 해당 worktree 경로 확인
- 새 worktree 생성
- 기존 worktree 열기
- 세션 시작 또는 재개
- 지시문 주입

구체 명령 구현은 환경에 따라 다를 수 있으므로 고정하지 않는다.

---

## Examples

### Example 1
현재 위치: `service-a/`
사용자 요청:  
"인증 리팩터링 시작해"

동작:
- 현재 repo 루트
- 적합한 기존 worktree 없음
- 새 worktree 생성
- branch: `feature/auth-refactor`
- 새 세션 시작
- 프롬프트 주입

### Example 2
현재 위치: `service-a-worktrees/feature-auth-refactor`
사용자 요청:  
"CI 깨진 거 수정해"

동작:
- 현재 worktree 확인
- 같은 기능의 후속 작업으로 판단
- 현재 worktree 유지
- 동일 세션에서 재개 또는 새 세션 연결

### Example 3
현재 위치: `service-a-worktrees/feature-auth-refactor`
사용자 요청:  
"PR 482 리뷰 P1 반영해"

동작:
- 현재 작업과 unrelated 가능성 평가
- 기존 `review/pr-482-p1-fixes` worktree 존재 시 재사용
- 없으면 새 worktree 생성

---

## Anti-Patterns

다음 행동은 피한다.

- repo 루트에서 곧바로 작업을 시작해 원본 작업공간을 오염시키기
- unrelated 작업을 같은 worktree에 계속 쌓기
- 브랜치명을 임시 이름으로 만들기
- 기존 worktree가 있음에도 새 worktree를 계속 중복 생성하기
- 사용자가 직접 low-level workmux 명령을 외워야 하는 UX를 강요하기

---

## Success Criteria

이 스킬이 잘 동작하면 다음이 가능해야 한다.

1. 사용자는 자연어로만 작업을 지시한다
2. 작업은 자동으로 적절한 worktree/session으로 라우팅된다
3. 같은 작업의 후속 작업은 같은 worktree로 귀속된다
4. 다른 작업은 자동으로 분리된다
5. 사용자는 workmux 명령을 거의 기억하지 않아도 된다
6. 단일 저장소 안에서도 여러 작업을 병렬로 안전하게 운영할 수 있다

---

## Future Extensions

향후 확장 가능 항목:

- GitHub PR/CI 이벤트 기반 자동 후속 작업 라우팅
- 기존 worktree 목적을 메타데이터 파일로 저장
- 세션별 요약/상태 저장
- 대시보드와 연계한 시각화
- planner/coder/reviewer 분업 모델과 연동
- ReproGate workflow runtime과 연결
