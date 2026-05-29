# auto

전체 파이프라인 오케스트레이터. 사용자의 자연어 요청을 받아 Step 1~10을 순서대로 실행하며 사용자 체크포인트 P·A·B를 관리하고, 분기·검증 루프·테스트 실패 처리를 제어한다.

## 입력

- 사용자의 자연어 요청 문자열 (예: `"login 함수에서 NPE 발생하는 버그 수정해줘"`)

## 실행 절차

### Step 1 — 요청 접수 및 세션 초기화

1. 현재 시각과 요청 키워드를 기반으로 `{session-id}`를 결정한다.
   - 형식: `YYYYMMDD-{작업키워드}` (소문자 + 하이픈 + 숫자만 사용)
   - 예: `20260521-fix-login-npe`, `20260521-review-auth-module`
2. `.coding-expert-artifacts/{session-id}/` 디렉토리를 생성한다.
3. 사용자 요청 원문을 `.coding-expert-artifacts/{session-id}/request.md`에 저장한다.

### Step 2 — 의도 분류

`classify` skill을 호출한다.
- 전달 인자: `session-id`

### Step 2.5 — 실행 계획 확인 (체크포인트 P)

`task-spec.md`를 읽어 아래 내용을 사용자에게 제시하고 승인을 받는다.

**제시 항목:**

1. **작업 유형** — 분류된 task-type과 한 줄 분류 근거 (`task-spec.md`의 `분류 근거` 필드)
2. **작업 범위** — 대상 파일·디렉토리 목록 (`task-spec.md`의 `target-files` 필드)
3. **실행 단계** — task-type과 `needs-architecture-plan` 값을 기반으로 각 단계의 실행 여부를 표시한다

```
✅ Step 1   — 요청 접수 (완료)
✅ Step 2   — 의도 분류 (완료)
▶  Step 2.5 — 실행 계획 확인  ← 체크포인트 P (지금 여기)
   Step 3   — 코드베이스 탐색
   Step 4   — 아키텍처 계획    [실행 예정] 또는 [건너뜀]
   Step 5   — 작업 수행         ({task-type}에 해당하는 worker 실행)
   Step 6   — 자가 검증
   Step 7   — 최종 확인         ← 체크포인트 B
   Step 8   — 변경 적용         [실행 예정] 또는 [건너뜀 — review]
   Step 9   — 자동 테스트       [실행 예정] 또는 [건너뜀 — review]
   Step 10  — Git 업로드        [실행 예정] 또는 [건너뜀 — review]
```

Step 4 표시 기준: `generate`이면 `[실행 예정 — 필수]`, `needs-architecture-plan: true`인 refactor·bugfix이면 `[실행 예정]`, 그 외에는 `[건너뜀]`.
Step 8~10 표시 기준: `review`이면 `[건너뜀]`, 그 외에는 `[실행 예정]`.

4. **예상 산출물** — task-type별 예상 출력물을 나열한다
   - review: 진단 리포트 (`work-result.md`)
   - bugfix: 원인 분석 + 수정 패치 + 회귀 방지 테스트 (`work-result.md`, `changes/`)
   - generate: 신규 코드 + 테스트 파일 (`work-result.md`, `changes/`)
   - refactor: 리팩토링 패치 (`work-result.md`, `changes/`)

사용자 응답에 따라 분기한다:
- **승인**: `plan-summary.md`에 확정된 계획을 기록하고 Step 3으로 진행한다.
- **수정 요청**: 수정 내용을 `task-spec.md`에 반영한 뒤 실행 계획을 다시 제시한다.
- **취소**: 파이프라인을 종료하고 산출물 디렉토리 경로를 안내한다.

### Step 3 — 코드베이스 탐색

`explore` skill을 호출한다.
- 전달 인자: `session-id`

### Step 4 — 아키텍처 계획 (조건부)

`task-spec.md`의 작업 유형을 확인한다.
- `review` 유형이면 이 단계를 건너뛴다.
- `generate` 유형이면 반드시 실행한다.
- `refactor` / `bugfix` 유형이면 폴더 구조 변경(파일 이동·신규 디렉토리 생성)이 동반될 때만 실행한다.

실행하는 경우: `design` skill을 호출한다.
- 전달 인자: `session-id`, `task-type`

사용자가 수정을 요청하면 `design` skill을 재호출한다.

### Step 5 — 작업 수행

`execute` skill을 호출한다.
- 전달 인자: `session-id`, `task-type`

### Step 6 — 자가 검증

`verify` skill을 호출한다.
- 전달 인자: `session-id`, `task-type`

verify skill 내부에서 REVISE 판정 시 `execute` skill이 1회 재호출된다. 재실행 후에도 REVISE이면 검증 결과를 그대로 유지하고 체크포인트 B에서 사용자에게 REVISE 상태임을 알리고 진행 여부를 확인한다.

### Step 7 — 사용자 확인 체크포인트 B

사용자에게 다음 항목을 제시하고 승인/수정/거부를 수령한다:

1. 작업 요약 (분류된 작업 유형, 대상 파일 목록, 작업 내용 한 줄 요약)
2. 확정된 기능별 폴더 구조 (아키텍처 계획이 실행된 경우)
3. 진단 결과 (`work-result.md` 핵심 내용)
4. 검증 결과 (`verification.md`의 PASS/REVISE 판정 및 체크리스트)
5. 적용될 변경 diff (`changes/` 디렉토리의 파일 내용 또는 unified diff)
6. Git 업로드 범위 선택: `commit-only` / `commit-push` / `commit-push-pr` 중 하나

사용자 응답에 따라 분기:
- **승인**: Step 8로 진행한다.
- **수정 요청**: 해당 부분만 Step 5(필요 시 Step 4)로 돌아가 재작업 후 Step 6~7을 다시 실행한다.
- **거부**: 파이프라인을 종료하고 산출물 디렉토리 경로를 안내한다.

> **review 유형 특이사항**: `changes/`가 비어 있으므로 "적용될 변경" 항목 없이 진단 리포트만 제시하고, 이 시점에 파이프라인을 종료한다. Step 8~10을 건너뛴다.

### Step 8 — 변경 적용

사용자 승인 후에만 실행한다.

`apply` skill을 호출한다.
- 전달 인자: `session-id`

### Step 9 — 자동 테스트

`test` skill을 호출한다.
- 전달 인자: `session-id`, `task-type`

테스트 실패 시: 강력 경고를 표시하고, 사용자가 "테스트 실패를 인지하고 업로드"를 명시적으로 승인해야만 Step 10으로 진행한다.

### Step 10 — Git 업로드

`ship` skill을 호출한다.
- 전달 인자: `session-id`, `git-scope` (Step 7에서 사용자가 선택한 값)

테스트가 실패한 상태로 진행하는 경우 Step 9에서 받은 사용자 명시 승인을 ship skill에 함께 전달한다.

## 출력

- `.coding-expert-artifacts/{session-id}/` — 세션 전체 산출물 디렉토리
- 최종 보고: 적용된 파일 목록, git 브랜치/커밋/PR URL (git 레포인 경우), 산출물 디렉토리 경로
