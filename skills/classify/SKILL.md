---
name: classify
description: 자연어 요청을 4가지 작업 유형으로 분류하고 spec-verifier로 검증한다. 모호한 요청은 사용자 질문으로 명확화한다. Step 2 담당.
---

Step 2 담당. 자연어 요청을 4가지 작업 유형(review/bugfix/generate/refactor)으로 분류하고 작업 범위를 명세화하여 `task-spec.md`를 산출한다. 분류 후 `spec-verifier`가 명세 정확성을 검증하며, 모호한 요청은 사용자에게 질문하여 명확화한 뒤 재분류한다.

## 입력

- `session-id`: 현재 세션 식별자

## 실행 절차

### 1단계 — 의도 분류

`intent-classifier` agent를 호출한다.
- 전달 인자: `session-id`

agent는 `.coding-expert-artifacts/{session-id}/request.md`를 직접 읽어 나머지 컨텍스트를 확보하고, `task-spec.md`를 직접 작성한다.

### 2단계 — 명세 검증 루프 (A구간)

`spec-verifier` agent를 호출한다.
- 전달 인자: `session-id`

agent는 `request.md`와 `task-spec.md`를 직접 읽어 분류 적절성을 검증한다. 모호한 항목이 있으면 사용자에게 질문하여 답변을 `task-spec.md`에 반영한 뒤 `spec-verification.md`를 작성한다.

판정에 따라 분기한다:

- **PASS**: 완료. explore로 진행한다.

- **CLARIFIED**: 사용자 답변이 반영된 최신 `task-spec.md`를 바탕으로 `intent-classifier` agent를 재호출한다.
  - 전달 인자: `session-id`
  - 재실행 후 `spec-verifier` agent를 1회 더 호출한다.
  - 이후 판정이 PASS 또는 CLARIFIED이면 완료. REVISE이면 아래 REVISE 처리를 따른다.

- **REVISE**: `spec-verification.md`의 재작업 지시사항을 함께 전달하여 `intent-classifier` agent를 재호출한다.
  - 전달 인자: `session-id`, `revise-instructions` (spec-verification.md 재작업 지시사항)
  - 재실행 후 `spec-verifier` agent를 1회 더 호출한다.
  - 이후에도 REVISE이면 사용자에게 현재 `task-spec.md`를 보여주고 진행 여부를 확인한다. (최대 2회 재실행, 무한 루프 방지)

classify skill이 단독 실행(`/classify "요청"`)된 경우, 검증 완료 후 `task-spec.md`와 `spec-verification.md` 내용을 사용자에게 요약 보고하고 종료한다.

## 출력

- `.coding-expert-artifacts/{session-id}/task-spec.md`
- `.coding-expert-artifacts/{session-id}/spec-verification.md`
