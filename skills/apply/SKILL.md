---
name: apply
description: 사용자 승인 후 changes/를 실제 소스에 적용하고, apply-verifier로 적용 결과를 검증한다. 예상외 수정 발견 시 사용자에게 확인을 요청한다. Step 8 담당.
---

Step 8 담당. 사용자 승인 후 `changes/`의 패치를 아키텍처 계획의 폴더 경로 그대로 실제 소스 파일에 적용하고, 적용 결과를 검증한다.

## 입력

- `session-id`: 현재 세션 식별자

## 실행 절차

### 1단계 — 변경 적용

`change-applier` agent를 호출한다.
- 전달 인자: `session-id`

agent는 `.coding-expert-artifacts/{session-id}/changes/`와 `architecture-plan.md`(존재하는 경우)를 직접 읽어 패치를 실제 소스 파일에 적용하고, `apply-log.md`를 직접 작성한다. `changes/`의 폴더 경로 구조를 보존하며 실제 소스에 적용한다.

### 2단계 — 적용 검증 (C구간)

`apply-verifier` agent를 호출한다.
- 전달 인자: `session-id`

agent는 `apply-log.md`, `architecture-plan.md`(존재하는 경우), `changes/`를 직접 읽어 적용 완결성과 범위 일탈 여부를 검증한다. 예상외 수정 파일이 발견되면 사용자에게 확인을 요청한 뒤 `apply-verification.md`를 작성한다.

판정에 따라 분기한다:

- **PASS** / **PASS(사용자확인)**: 완료. test로 진행한다.

- **ISSUES**: 파이프라인을 중단한다. `apply-verification.md`의 문제 내용과 권장 조치를 사용자에게 보고하고 수동 확인을 안내한다. test·ship 단계는 실행하지 않는다.

apply skill이 단독 실행(`/apply`)된 경우, 검증 완료 후 `apply-log.md`와 `apply-verification.md` 기반으로 적용된 파일 목록, 결과, 검증 판정을 사용자에게 보고하고 종료한다.

## 출력

- `.coding-expert-artifacts/{session-id}/apply-log.md`
- `.coding-expert-artifacts/{session-id}/apply-verification.md`
- 실제 소스 파일 변경
