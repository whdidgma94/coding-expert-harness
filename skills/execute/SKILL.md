# execute

Step 5 담당. `task-spec.md`의 작업 유형에 따라 해당 worker agent 1개만 분기 호출하여 작업 결과물을 생성한다.

## 입력

- `session-id`: 현재 세션 식별자
- `task-type`: 작업 유형 (review / bugfix / generate / refactor)
- `verification-feedback` (선택): verify skill이 REVISE 판정 시 전달하는 재작업 지시사항

## 실행 절차

`task-type` 값에 따라 해당 worker agent 1개만 호출한다. 4개를 병렬 실행하지 않는다.

- `review` → `code-reviewer` agent 호출
  - 전달 인자: `session-id`, `verification-feedback` (재실행 시)

- `bugfix` → `bug-fixer` agent 호출
  - 전달 인자: `session-id`, `verification-feedback` (재실행 시)

- `generate` → `code-generator` agent 호출
  - 전달 인자: `session-id`, `verification-feedback` (재실행 시)

- `refactor` → `refactorer` agent 호출
  - 전달 인자: `session-id`, `verification-feedback` (재실행 시)

각 agent는 `session-id`를 받아 `.coding-expert-artifacts/{session-id}/task-spec.md`, `codebase-context.md`, `architecture-plan.md`(존재하는 경우)를 직접 읽어 나머지 컨텍스트를 확보하고, `work-result.md`와 `changes/`를 직접 작성한다.

execute skill이 단독 실행(`/execute`)된 경우, 작업 완료 후 `work-result.md` 핵심 내용을 사용자에게 요약 보고하고 종료한다.

## 출력

- `.coding-expert-artifacts/{session-id}/work-result.md`
- `.coding-expert-artifacts/{session-id}/changes/` (review 유형 제외)
