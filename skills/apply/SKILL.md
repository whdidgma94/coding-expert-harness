---
name: apply
description: Step 7 담당. 사용자가 /apply 커맨드를 실행하거나 auto skill이 Step 6 사용자 승인 이후 호출할 때 실행된다. change-applier로 파일을 적용한 뒤 git-operator로 git 작업(브랜치·커밋·push·PR)을 수행한다.
---

# apply skill

`changes/` 디렉토리의 패치를 실제 소스 파일에 적용하고, git 작업을 수행한다. 사용자의 명시적 승인 이후에만 실행된다.

## 입력

- `{session-id}`: 현재 세션 식별자
- `.coding-expert-artifacts/{session-id}/changes/`: 적용할 패치 파일 모음
- `.coding-expert-artifacts/{session-id}/codebase-context.md`: git 여부 및 현재 브랜치 정보
- `.coding-expert-artifacts/{session-id}/task-spec.md`: 작업 유형 및 작업 범위

---

## Step 1 — 사전 조건 확인

1. `changes/` 디렉토리가 존재하고 파일이 1개 이상인지 확인한다.
2. 파일이 없으면 (review 유형) "적용할 변경이 없습니다. review 유형은 changes/가 비어 있어 apply를 실행하지 않습니다." 메시지를 출력하고 종료한다.

---

## Step 2 — 파일 적용 (change-applier)

**sub-agent를 호출**하여 change-applier agent를 실행한다.

전달 인자:
- `session-id`
- `confirmed-paths` (generate 유형 한정 — Step 6에서 사용자가 확정한 신규 파일 경로)

change-applier는 `changes/`의 패치를 실제 소스 파일에 적용하고 apply-log.md 초안을 작성한다.
git 브랜치·커밋·push는 수행하지 않는다.

---

## Step 3 — git 작업 (git-operator)

`codebase-context.md`에서 git 레포 여부를 확인한다.

### git 레포인 경우

사용자에게 다음 git 작업 범위를 확인한다:

**사용자 확인 체크포인트**
- 커밋만 할지, push까지 할지, PR 생성까지 할지 선택받는다.
  - `commit-only` — 로컬 브랜치 생성 + 커밋만
  - `commit-push` — 커밋 + 원격 push
  - `commit-push-pr` — 커밋 + push + GitHub PR 생성

**sub-agent를 호출**하여 git-operator agent를 실행한다.

전달 인자:
- `session-id`
- `task-type`: task-spec.md에서 읽은 작업 유형
- `task-summary`: work-result.md에서 추출한 한 줄 요약
- `operation`: 위 체크포인트에서 사용자가 선택한 값
- `base-branch`: 기본값 `main` (사용자가 지정한 경우 해당 브랜치)
- `pr-visibility`: PR 생성 시 draft 또는 ready (기본값 `ready`)

### git 레포가 아닌 경우

git-operator를 호출하지 않는다. Step 2의 직접 파일 수정으로 완료된 것으로 처리한다.

---

## Step 4 — 적용 결과 보고

apply-log.md를 기반으로 다음 내용을 사용자에게 보고한다:

- 적용된 파일 목록 및 결과
- **git 레포인 경우**: 브랜치명, 커밋 해시, push 결과, PR URL (생성한 경우)
- **git 없는 경우**: 수정된 파일 목록
- 실패 항목이 있으면 원인과 수동 처리 방법을 안내한다.

---

## 출력

- `.coding-expert-artifacts/{session-id}/apply-log.md` — 파일 적용 및 git 작업 결과
- 실제 소스 파일 변경
- (git 레포) 신규 브랜치 + 커밋 + push + PR (선택한 operation에 따라)
