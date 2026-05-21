---
name: execute
description: Step 4 담당. auto skill이 Step 4에서 호출하거나 사용자가 /execute 커맨드를 실행할 때 호출된다. task-spec.md의 작업 유형에 따라 review/bugfix/generate/refactor 중 하나의 worker agent를 단독 실행하여 work-result.md와 changes/를 산출한다.
---

# execute skill

`task-spec.md`의 작업 유형을 읽어 해당하는 worker agent 1개만 분기 실행하여 작업 결과물을 생성한다. 4가지 worker agent를 병렬 실행하지 않는다.

## 입력

- `{session-id}`: 현재 세션 식별자
- `.coding-expert-artifacts/{session-id}/task-spec.md`: 작업 유형, 대상 파일, 완료 기준
- `.coding-expert-artifacts/{session-id}/codebase-context.md`: 코드베이스 컨텍스트
- `{revision-instructions}` (선택, verify skill이 재호출 시 전달): `verification.md`의 REVISE 지시사항

---

## Step 1 — 작업 유형 확인

`task-spec.md`에서 작업 유형을 읽는다.

- `review` → Step 2-A 실행
- `bugfix` → Step 2-B 실행
- `generate` → Step 2-C 실행
- `refactor` → Step 2-D 실행

revision-instructions가 전달된 경우, 해당 지시사항을 worker agent에게 추가 입력으로 전달한다.

---

## Step 2-A — review 유형: code-reviewer agent 호출

**sub-agent를 호출**하여 code-reviewer agent를 실행한다.

agent에게 전달하는 인자:
- `session-id`: {실제 session-id 값}
- `revision-notes` (REVISE 재실행인 경우): {verification.md에서 추출한 재작업 지시사항}

agent는 `task-spec.md`와 `codebase-context.md`를 직접 읽어 나머지 컨텍스트를 확보한다.

**출력**: `.coding-expert-artifacts/{session-id}/work-result.md` (진단 리포트)

---

## Step 2-B — bugfix 유형: bug-fixer agent 호출

**sub-agent를 호출**하여 bug-fixer agent를 실행한다.

agent에게 전달하는 인자:
- `session-id`: {실제 session-id 값}
- `revision-notes` (REVISE 재실행인 경우): {verification.md에서 추출한 재작업 지시사항}

agent는 `task-spec.md`와 `codebase-context.md`를 직접 읽어 나머지 컨텍스트를 확보한다.

**출력**: `.coding-expert-artifacts/{session-id}/work-result.md`, `.coding-expert-artifacts/{session-id}/changes/`

---

## Step 2-C — generate 유형: code-generator agent 호출

**sub-agent를 호출**하여 code-generator agent를 실행한다.

agent에게 전달하는 인자:
- `session-id`: {실제 session-id 값}
- `revision-notes` (REVISE 재실행인 경우): {verification.md에서 추출한 재작업 지시사항}

agent는 `task-spec.md`와 `codebase-context.md`를 직접 읽어 나머지 컨텍스트를 확보한다.

**출력**: `.coding-expert-artifacts/{session-id}/work-result.md`, `.coding-expert-artifacts/{session-id}/changes/`

---

## Step 2-D — refactor 유형: refactorer agent 호출

**sub-agent를 호출**하여 refactorer agent를 실행한다.

agent에게 전달하는 인자:
- `session-id`: {실제 session-id 값}
- `revision-notes` (REVISE 재실행인 경우): {verification.md에서 추출한 재작업 지시사항}

agent는 `task-spec.md`와 `codebase-context.md`를 직접 읽어 나머지 컨텍스트를 확보한다.

**출력**: `.coding-expert-artifacts/{session-id}/work-result.md`, `.coding-expert-artifacts/{session-id}/changes/`

---

## Step 3 — 산출물 존재 확인

- `work-result.md`가 생성되었는지 확인한다.
- review 유형이 아닌 경우 `changes/` 디렉토리에 최소 1개 파일이 있는지 확인한다.
- 산출물이 없으면 해당 worker agent를 재호출한다.

---

## 출력

- `.coding-expert-artifacts/{session-id}/work-result.md`
- `.coding-expert-artifacts/{session-id}/changes/` (review 유형 제외)
