# verify

Step 6 담당. 작업 결과를 코드베이스 컨벤션·요청 충족·기능별 폴더 구조 준수·회귀 위험 관점에서 정적으로 검증하고 PASS/REVISE를 판정하여 `verification.md`를 산출한다. REVISE 판정 시 `execute` skill을 1회 재호출한다.

## 입력

- `session-id`: 현재 세션 식별자
- `task-type`: 작업 유형 (review / bugfix / generate / refactor)

## 실행 절차

### 1차 검증

`work-verifier` agent를 호출한다.
- 전달 인자: `session-id`, `task-type`

agent는 `task-spec.md`, `codebase-context.md`, `work-result.md`, `changes/`를 직접 읽어 검증하고, `verification.md`를 직접 작성한다.

### 판정 분기

`verification.md`의 판정을 확인한다.

- **PASS**: 완료. auto skill의 Step 7 체크포인트 B로 진행한다.
- **REVISE**: `execute` skill을 재호출한다 (1회 한정).
  - 전달 인자: `session-id`, `task-type`, `verification-feedback` (verification.md에서 추출한 재작업 지시사항)
  - execute skill 완료 후 `work-verifier` agent를 재호출한다.
    - 전달 인자: `session-id`, `task-type`
    - agent는 갱신된 파일들을 직접 읽어 `verification.md`를 덮어쓴다.

### 재시도 후 분기

재실행 후에도 REVISE 판정이면: 검증 결과를 그대로 유지하고 종료한다. auto skill이 Step 7 체크포인트 B에서 사용자에게 REVISE 상태임을 알리고 진행 여부를 확인한다 (무한 루프 방지, 최대 1회 재시도).

## 출력

- `.coding-expert-artifacts/{session-id}/verification.md`
