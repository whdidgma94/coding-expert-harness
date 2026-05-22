# apply

Step 8 담당. 사용자 승인 후 `changes/`의 패치를 아키텍처 계획의 폴더 경로 그대로 실제 소스 파일에 적용한다.

## 입력

- `session-id`: 현재 세션 식별자

## 실행 절차

`change-applier` agent를 호출한다.
- 전달 인자: `session-id`

agent는 `.coding-expert-artifacts/{session-id}/changes/`와 `architecture-plan.md`(존재하는 경우)를 직접 읽어 패치를 실제 소스 파일에 적용하고, `apply-log.md`를 직접 작성한다. agent는 `changes/`의 폴더 경로 구조를 보존하며 실제 소스에 적용한다.

apply skill이 단독 실행(`/apply`)된 경우, 적용 완료 후 `apply-log.md` 기반으로 적용된 파일 목록과 결과를 사용자에게 보고하고 종료한다.

## 출력

- `.coding-expert-artifacts/{session-id}/apply-log.md`
- 실제 소스 파일 변경
