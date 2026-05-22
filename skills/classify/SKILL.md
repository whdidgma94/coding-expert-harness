# classify

Step 2 담당. 자연어 요청을 4가지 작업 유형(review/bugfix/generate/refactor)으로 분류하고 작업 범위를 명세화하여 `task-spec.md`를 산출한다.

## 입력

- `session-id`: 현재 세션 식별자

## 실행 절차

`intent-classifier` agent를 호출한다.
- 전달 인자: `session-id`

agent는 `.coding-expert-artifacts/{session-id}/request.md`를 직접 읽어 나머지 컨텍스트를 확보하고, `task-spec.md`를 직접 작성한다.

classify skill이 단독 실행(`/classify "요청"`)된 경우, 생성된 `task-spec.md` 내용을 사용자에게 요약 보고하고 종료한다.

## 출력

- `.coding-expert-artifacts/{session-id}/task-spec.md`
