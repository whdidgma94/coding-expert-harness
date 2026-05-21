---
name: auto
description: 전체 파이프라인 오케스트레이터. 사용자가 /auto "요청" 커맨드를 실행할 때 호출되며, classify → explore → execute → verify → 사용자 확인 → apply 순으로 전체 파이프라인을 실행한다.
---

# auto skill

사용자의 자연어 요청을 받아 코드 리뷰·버그 수정·코드 생성·리팩토링 전체 파이프라인을 순서대로 실행하는 오케스트레이터 skill이다.

## 입력

- 사용자의 자연어 요청 문자열 (예: `"login 함수에서 NPE 발생하는 버그 수정해줘"`)

---

## Step 1 — 요청 접수 및 세션 초기화

1. 사용자 요청 원문을 수신한다.
2. 현재 시각과 요청 키워드를 기반으로 `{session-id}`를 결정한다.
   - 형식: `YYYYMMDD-{작업키워드}` (소문자 + 하이픈 + 숫자만 사용)
   - 예: `20260521-fix-login-npe`, `20260521-review-auth-module`
3. 산출물 디렉토리를 생성한다: `.coding-expert-artifacts/{session-id}/`
4. 사용자 요청 원문을 `.coding-expert-artifacts/{session-id}/request.md`에 저장한다.

**출력**: `{session-id}`, `.coding-expert-artifacts/{session-id}/request.md`

---

## Step 2 — 의도 분류 (classify skill 호출)

**classify skill을 호출**하여 요청을 작업 유형으로 분류하고 `task-spec.md`를 생성한다.

- 입력: `{session-id}`, 사용자 요청 원문
- 출력: `.coding-expert-artifacts/{session-id}/task-spec.md`

---

## Step 3 — 코드베이스 탐색 (explore skill 호출)

**explore skill을 호출**하여 대상 코드베이스를 탐색하고 `codebase-context.md`를 생성한다.

- 입력: `{session-id}`, `task-spec.md`
- 출력: `.coding-expert-artifacts/{session-id}/codebase-context.md`

---

## Step 4 — 작업 수행 (execute skill 호출)

**execute skill을 호출**하여 `task-spec.md`의 작업 유형에 따라 작업을 수행한다.

- 입력: `{session-id}`, `task-spec.md`, `codebase-context.md`
- 출력: `.coding-expert-artifacts/{session-id}/work-result.md`, `.coding-expert-artifacts/{session-id}/changes/`

---

## Step 5 — 자가 검증 (verify skill 호출)

**verify skill을 호출**하여 작업 결과를 검증하고 PASS/REVISE를 판정한다.

- 입력: `{session-id}`, `task-spec.md`, `codebase-context.md`, `work-result.md`, `changes/`
- 출력: `.coding-expert-artifacts/{session-id}/verification.md`

verify skill 내부에서 REVISE 판정 시 execute skill이 1회 재호출되며, 재실행 후에도 REVISE면 검증 결과를 그대로 다음 단계로 넘긴다.

---

## Step 6 — 사용자 확인 체크포인트

**사용자 확인 체크포인트**

다음 정보를 사용자에게 제시한다:

1. **작업 요약**: 분류된 작업 유형, 대상 파일 목록, 작업 내용 한 줄 요약
2. **진단 결과**: `work-result.md` 핵심 내용
3. **검증 결과**: `verification.md`의 PASS/REVISE 판정 및 체크리스트
4. **적용될 변경(diff)**: `changes/` 디렉토리의 각 변경 파일 내용 또는 unified diff
5. **신규 파일 경로 확인** (generate 유형에 해당하는 경우): code-generator가 제안한 경로를 명시하고 사용자 확인을 받는다.

사용자 응답에 따라 분기한다:
- **승인**: Step 7로 진행한다.
- **수정 요청**: 수정 내용을 반영하여 Step 4(execute skill)로 돌아가 재작업 후 Step 5~6을 다시 실행한다.
- **거부**: 파이프라인을 종료하고 산출물 디렉토리 경로를 안내한다.

> **review 유형 특이사항**: `changes/`가 비어 있으므로 "적용될 변경" 항목 없이 진단 리포트만 제시하고, Step 7을 건너뛰어 파이프라인을 종료한다.

---

## Step 7 — 변경 적용 (apply skill 호출)

사용자 승인 후에만 실행한다.

**apply skill을 호출**하여 `changes/`의 패치를 실제 소스 파일에 적용한다.

- 입력: `{session-id}`, `changes/`, `codebase-context.md`
- 출력: `.coding-expert-artifacts/{session-id}/apply-log.md`, 실제 소스 파일 변경

---

## 파이프라인 종료

apply skill 실행이 완료되면 사용자에게 다음을 보고한다:

- 적용된 파일 목록 (`apply-log.md` 기반)
- git 브랜치 정보 (git 레포인 경우)
- 산출물 디렉토리 경로: `.coding-expert-artifacts/{session-id}/`
