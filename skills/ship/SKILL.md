# ship

Step 10 담당. 사용자가 체크포인트 B에서 선택한 범위(`commit-only` / `commit-push` / `commit-push-pr`)로 브랜치 생성·커밋·push·PR 생성을 수행한다.

## 입력

- `session-id`: 현재 세션 식별자
- `git-scope`: Git 업로드 범위 (`commit-only` / `commit-push` / `commit-push-pr`)

## 실행 절차

`git-operator` agent를 호출한다.
- 전달 인자: `session-id`, `git-scope`

agent는 `.coding-expert-artifacts/{session-id}/apply-log.md`, `task-spec.md`, `work-result.md`를 직접 읽어 브랜치명·커밋 메시지·PR 제목을 결정하고, `git-scope`에 따라 git 작업을 수행한 뒤 `apply-log.md`에 git 작업 결과를 추가 기록한다.

ship skill이 단독 실행(`/ship`)된 경우, `apply-log.md` 기반으로 브랜치·커밋 해시·push 결과·PR URL을 사용자에게 보고하고 종료한다.

## 출력

- `.coding-expert-artifacts/{session-id}/apply-log.md` (git 작업 결과 추가 기록)
