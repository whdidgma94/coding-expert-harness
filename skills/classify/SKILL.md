---
name: classify
description: Step 2 담당. 사용자가 /classify "요청" 커맨드를 실행하거나 auto skill이 Step 2에서 호출할 때 실행된다. 자연어 요청을 4가지 작업 유형(review/bugfix/generate/refactor)으로 분류하고 작업 범위를 명세화하여 task-spec.md를 산출한다.
---

# classify skill

사용자의 자연어 요청을 분석하여 작업 유형을 결정하고, 작업 범위·대상 파일 후보·요청 충족 기준을 명세화한 `task-spec.md`를 생성한다.

## 입력

- `{session-id}`: 현재 세션 식별자
- 사용자 자연어 요청 원문 (`.coding-expert-artifacts/{session-id}/request.md` 또는 직접 전달)

---

## Step 1 — 산출물 디렉토리 확인

1. `.coding-expert-artifacts/{session-id}/` 디렉토리가 존재하는지 확인한다.
2. 존재하지 않으면 생성하고 `request.md`에 요청 원문을 저장한다.
3. 이미 존재하면 `request.md`를 읽어 요청 원문을 확인한다.

---

## Step 2 — intent-classifier agent 호출

**sub-agent를 호출**하여 intent-classifier agent를 실행한다.

agent에게 전달하는 인자:
- `session-id`: {실제 session-id 값}

agent는 `.coding-expert-artifacts/{session-id}/request.md`를 직접 읽어 나머지 컨텍스트를 확보한다.

**출력**: `.coding-expert-artifacts/{session-id}/task-spec.md`

---

## Step 3 — task-spec.md 구조 검증

생성된 `task-spec.md`가 다음 항목을 포함하는지 확인한다:

- `작업 유형`: review / bugfix / generate / refactor 중 하나
- `대상 파일 후보`: 파일 경로 목록 또는 "탐색 후 결정"
- `작업 범위`: 요청의 핵심 내용 요약
- `완료 기준`: 요청이 충족되었음을 판단할 수 있는 구체적인 기준 목록

항목이 누락된 경우 intent-classifier agent를 다시 호출하여 보완한다.

---

## 출력

- `.coding-expert-artifacts/{session-id}/task-spec.md` — 분류 결과 및 작업 명세

classify skill이 단독 실행(`/classify "요청"`)된 경우, 생성된 `task-spec.md` 내용을 사용자에게 요약 보고하고 종료한다.
