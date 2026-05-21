---
name: verify
description: Step 5 담당. auto skill이 Step 5에서 호출할 때 실행된다. 작업 결과(work-result.md, changes/)를 코드베이스 컨벤션·요청 충족·회귀 위험 관점에서 검증하여 PASS/REVISE를 판정하고 verification.md를 산출한다. REVISE 판정 시 execute skill을 1회 재호출한다.
---

# verify skill

execute skill이 산출한 작업 결과를 다각도로 검증하여 품질을 보장한다. 최대 1회 재시도 루프를 포함한다.

## 입력

- `{session-id}`: 현재 세션 식별자
- `.coding-expert-artifacts/{session-id}/task-spec.md`
- `.coding-expert-artifacts/{session-id}/codebase-context.md`
- `.coding-expert-artifacts/{session-id}/work-result.md`
- `.coding-expert-artifacts/{session-id}/changes/`

---

## Step 1 — work-verifier agent 호출 (1차)

**sub-agent를 호출**하여 work-verifier agent를 실행한다.

agent에게 전달하는 인자:
- `session-id`: {실제 session-id 값}
- `attempt`: `1`

agent는 `task-spec.md`, `codebase-context.md`, `work-result.md`, `changes/`를 직접 읽어 나머지 컨텍스트를 확보한다.

**출력**: `.coding-expert-artifacts/{session-id}/verification.md`

---

## Step 2 — 판정 분기

`verification.md`의 판정을 읽는다.

- **PASS** → Step 4(완료)로 이동한다.
- **REVISE** → Step 3(재시도)으로 이동한다.

---

## Step 3 — execute skill 재호출 (1회 한정)

> 이 단계는 1회만 실행된다. 이미 재시도를 한 경우 Step 3을 건너뛰고 Step 4로 진행한다.

`verification.md`에서 재작업 지시사항(`revision-instructions`)을 추출한다.

**execute skill을 호출**하여 execute skill을 실행한다.

- 인자: `session-id`, `revision-instructions` (위에서 추출한 값)
- execute skill이 완료되면 work-result.md와 changes/가 갱신된다.

갱신된 산출물로 **sub-agent를 호출**하여 work-verifier agent를 재실행한다.

- 인자: `session-id`, `attempt`: `2`
- agent는 갱신된 파일들을 직접 읽는다. `verification.md`를 덮어써서 최신 검증 결과를 기록한다.

---

## Step 4 — 완료

재시도 후에도 REVISE 판정인 경우 검증 결과를 그대로 유지하고 종료한다. auto skill이 Step 6 체크포인트에서 사용자에게 REVISE 상태임을 알리고 진행 여부를 묻는다. (무한 루프 방지 — 최대 1회 재시도)

---

## 출력

- `.coding-expert-artifacts/{session-id}/verification.md` — PASS/REVISE 판정, 체크리스트, 재시도 지시사항(REVISE 시)
