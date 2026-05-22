# test

Step 9 담당. 적용된 실제 소스에서 `codebase-context.md`의 표준 테스트 명령을 자동 실행하여 결과를 `test-report.md`에 산출한다.

## 입력

- `session-id`: 현재 세션 식별자
- `task-type`: 작업 유형 (review / bugfix / generate / refactor)

## 실행 절차

`task-type`이 `review`이면 이 skill을 실행하지 않고 즉시 종료한다.

`test-runner` agent를 호출한다.
- 전달 인자: `session-id`, `task-type`

agent는 `codebase-context.md`를 직접 읽어 표준 테스트 명령을 확인하고 실행한 뒤, `test-report.md`를 직접 작성한다. generate 유형에서 worker가 생성한 테스트 파일도 함께 실행한다.

test skill이 단독 실행(`/test`)된 경우, `test-report.md` 기반으로 통과/실패 결과를 사용자에게 요약 보고하고 종료한다.

## 출력

- `.coding-expert-artifacts/{session-id}/test-report.md`
