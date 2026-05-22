# design

Step 4 담당. 작업 결과물의 기능별 폴더 구조·모듈 경계·신규/수정 파일 경로를 설계하여 `architecture-plan.md`를 산출하고, 사용자 확인 체크포인트 A를 수행한다.

## 입력

- `session-id`: 현재 세션 식별자
- `task-type`: 작업 유형 (generate / refactor / bugfix)

## 실행 절차

`architecture-planner` agent를 호출한다.
- 전달 인자: `session-id`, `task-type`

agent는 `.coding-expert-artifacts/{session-id}/task-spec.md`와 `.coding-expert-artifacts/{session-id}/codebase-context.md`를 직접 읽어 나머지 컨텍스트를 확보하고, `architecture-plan.md`를 직접 작성한다. agent는 `architecture-plan.md` 작성 후 사용자 확인 체크포인트 A를 직접 수행한다 (폴더 트리·모듈 경계·신규/수정 파일 경로 제시 및 승인/수정 수령).

사용자가 수정을 요청하면 `architecture-planner` agent를 재호출하여 `architecture-plan.md`를 갱신한다.
- 전달 인자: `session-id`, `task-type`

design skill이 단독 실행(`/design`)된 경우, 체크포인트 A 완료 후 `architecture-plan.md` 경로를 사용자에게 안내하고 종료한다.

## 출력

- `.coding-expert-artifacts/{session-id}/architecture-plan.md`
