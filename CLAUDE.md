# coding-expert-harness

`coding-expert-harness`는 개발자가 자연어 프롬프트로 개발 업무를 요청하면 이를 분석·분류하여 코드 리뷰·버그 수정·코드 생성·리팩토링 4가지 작업 중 적합한 흐름으로 자동 라우팅하고 실행하는 범용 코딩 보조 하네스다. 핵심은 (1) **기능별 폴더 구조 설계** — 코드를 생성·확장할 때 관련 기능을 같은 폴더에 모으는 계층 구조를 먼저 설계하고 사용자와 확정한 뒤에야 코드를 생성한다, (2) **아키텍처 계획 단계** — 폴더 트리·모듈 경계·파일 경로를 사용자 체크포인트 A에서 승인받는다, (3) **자동 테스트 실행** — 변경 적용 후 표준 테스트를 자동 실행하고 테스트가 없는 프로젝트에서는 테스트 파일을 함께 생성한다, (4) **완결적 Git 통합** — 사용자 최종 승인 후 브랜치 생성→커밋→push→PR 생성까지 자동 수행한다는 4가지다. 모든 작업 결과물은 산출물 디렉토리에 먼저 기록하고 사용자 승인을 받은 뒤에만 실제 소스 파일에 적용한다.

## 산출물 경로

모든 작업 산출물은 `.coding-expert-artifacts/{session-id}/` 하위에 저장한다.
`{session-id}`는 요청 접수 시각과 작업 키워드 기반으로 결정한다 (예: `20260521-add-auth-feature`). 소문자 + 하이픈 + 숫자만 사용한다.

산출물 목록:
- `request.md` — 사용자의 원문 자연어 요청 보존 (Step 1)
- `task-spec.md` — 분류된 작업 유형(review/bugfix/generate/refactor), 작업 범위, 대상 파일 후보, 요청 충족 기준 (Step 2 / intent-classifier)
- `spec-verification.md` — A구간 명세 검증 결과(PASS/CLARIFIED/REVISE), 모호성 질문·사용자 답변 이력 (Step 2 / spec-verifier)
- `plan-summary.md` — 체크포인트 P에서 사용자에게 제시된 실행 계획과 승인 결과 기록 (Step 2.5)
- `codebase-context.md` — 대상 코드베이스의 언어·프레임워크·기존 디렉토리 구조·기능별 폴더 패턴·코딩 컨벤션·테스트 디렉토리 존재 여부·표준 테스트/빌드 명령 (Step 3 / codebase-explorer)
- `architecture-plan.md` — 작업 결과물의 기능별 폴더 트리, 모듈 경계, 신규/수정 파일 경로 목록, 각 파일의 책임 요약. 사용자 체크포인트 A에서 승인된 최종본 (Step 4 / architecture-planner)
- `work-result.md` — 작업 수행 결과 요약. review는 진단 리포트, bugfix는 원인 분석, generate/refactor는 변경 의도와 파일 목록 (Step 5 / worker agents)
- `changes/` — 실제 소스에 적용될 변경 단위 모음. `architecture-plan.md`의 기능별 폴더 구조를 그대로 미러링한 하위 경로에 파일별 신규 내용 또는 unified diff 패치를 배치한다. review 유형에서는 비어 있다 (Step 5)
- `verification.md` — 자가 검증 결과(PASS/REVISE), 체크리스트, REVISE 시 재실행 지시사항 (Step 6 / work-verifier)
- `test-report.md` — 실행한 테스트 명령, 통과/실패 결과, 실패 케이스, 로그 요약 (Step 9 / test-runner)
- `apply-log.md` — 실제 소스에 적용된 파일 목록·적용 결과(Step 8 / change-applier) 및 git 작업 결과(브랜치·커밋 해시·push·PR URL) (Step 10 / git-operator)
- `apply-verification.md` — C구간 적용 검증 결과(PASS/PASS(사용자확인)/ISSUES), 예상외 수정 파일 및 사용자 확인 이력 (Step 8 / apply-verifier)

## 기능별 폴더 구조 원칙

코드를 생성·확장할 때 관련 기능을 같은 폴더에 모으는 feature 단위 계층 구조를 사용한다. 기존 코드베이스의 구조 패턴을 우선 따르되, 기능별로 폴더를 묶은 트리를 먼저 설계한다. 폴더 구조는 코드 생성 전 **사용자 확인 체크포인트 A**에서 확정된다. `changes/` 디렉토리는 `architecture-plan.md`의 폴더 트리를 그대로 미러링한 경로로 파일을 작성한다.

## 파이프라인 규칙

1. **소스 쓰기 범위 제한**: Step 1~6의 모든 agent는 `.coding-expert-artifacts/{session-id}/` 산출물 디렉토리에만 쓰기를 수행한다. 실제 소스 파일은 절대 수정하지 않는다. 실제 소스 수정은 Step 8의 `change-applier`만 수행하고, git 작업은 Step 10의 `git-operator`만 수행하며, 두 agent 모두 사용자 최종 승인(Step 7 체크포인트 B) 이후에만 실행된다.
2. **사용자 확인 체크포인트 A** (Step 4): generate 유형에서 항상 실행, refactor·bugfix 유형에서는 폴더 구조 변경이 동반될 때만 실행, review 유형에서는 건너뛴다. 체크포인트 A에서 폴더 트리·모듈 경계·신규/수정 파일 경로를 제시하고 승인을 받아야만 Step 5로 진행한다.
3. **사용자 확인 체크포인트 P** (Step 2.5): 의도 분류 완료 직후, 코드베이스 탐색 전에 실행한다. 작업 유형·분류 근거·대상 파일·실행 단계 목록(건너뛸 단계 포함)·예상 산출물을 제시하고 승인을 받는다. 승인 없이는 Step 3으로 진행하지 않는다. 수정 요청 시 task-spec.md를 갱신하고 계획을 재제시한다. 취소 시 파이프라인을 종료한다. `plan-summary.md`에 확정된 계획과 승인 결과를 기록한다.
4. **사용자 확인 체크포인트 B** (Step 7): 작업 요약·진단 결과·확정된 폴더 구조·적용될 diff·검증 결과를 제시하고 승인/수정/거부를 받는다. 이 승인 없이는 Step 8(실제 소스 적용)·Step 9(테스트)·Step 10(Git)으로 진행하지 않는다. Git 업로드 범위(`commit-only` / `commit-push` / `commit-push-pr`)도 함께 선택한다.
5. **A구간 검증 루프** (Step 2): classify 직후 spec-verifier가 task-spec.md의 분류 적절성을 검증한다. 요청이 모호하면 사용자에게 구체화 질문을 제시하고 답변을 반영하여 intent-classifier를 재실행한다 (최대 2회 재실행). 가장 저렴한 지점에서 방향 오류를 차단한다.
6. **C구간 검증 루프** (Step 8): apply 직후 apply-verifier가 적용 완결성과 범위 일탈 여부를 검증한다. 예상외 수정 파일이 발견되면 사용자에게 확인을 요청한다. ISSUES 판정 시 파이프라인을 중단하고 수동 확인을 안내한다.
7. **모호한 요청 구체화**: A구간에서 요청의 작업 유형·범위·전제 조건이 불명확한 경우 spec-verifier가 사용자에게 선택지 형식의 질문을 제시한다 (최대 3회). 답변을 반영하여 task-spec.md를 갱신한 뒤 재분류한다.
8. **기존 검증 루프**: Step 6에서 work-verifier가 REVISE 판정 시 `verification.md`의 지시사항을 입력으로 Step 5를 1회 재실행한다. 재실행 후에도 REVISE면 검증 결과를 사용자에게 보여주고 진행 여부를 묻는다 (최대 1회 재시도, 무한 루프 방지).
9. **자동 테스트 루프**: Step 9에서 test-runner가 실제 소스에 표준 테스트를 실행한다. 실패 시 `test-report.md`에 실패 내역을 기록하고 강력 경고를 표시한다. 사용자가 "테스트 실패를 인지하고 업로드"를 명시적으로 승인해야만 Step 10으로 진행한다.
10. **분기 실행**: Step 5는 `task-spec.md`의 작업 유형에 따라 worker agent(code-reviewer / bug-fixer / code-generator / refactorer) 1개만 실행한다. 4개를 병렬 실행하지 않는다.
11. **review 유형 종료 처리**: 작업 유형이 review인 경우 `changes/`가 비어 있으므로 Step 8(적용)·Step 9(테스트)·Step 10(Git)을 건너뛰고, Step 7에서 진단 리포트를 전달하는 것으로 종료한다.
12. **Git 통합**: Step 10에서 git-operator는 Step 7에서 선택한 범위로 동작한다. 브랜치명은 `coding-expert/{session-id}`, 커밋 메시지는 `{task-type}: {task-summary}` 형식. git 레포가 아니면 "git 레포 아님" 기록, remote 없으면 커밋까지만, `gh` 미인증이면 push까지만 수행한다. `git push --force`·`git reset --hard`는 절대 수행하지 않는다.

## 커맨드

```
/auto "요청"     ← 자연어 요청을 받아 분류→탐색→설계→수행→검증→적용→테스트→Git 업로드 전체 파이프라인 실행
/classify "요청" ← 요청을 작업 유형으로 분류만 수행 (task-spec.md 산출)
/design          ← 승인된 task-spec.md 기반으로 기능별 폴더 구조 설계만 실행 (architecture-plan.md 산출)
/execute         ← 승인된 architecture-plan.md 기반으로 작업 수행만 실행
/apply           ← 검증·승인된 changes/ 를 실제 소스에 적용
/test            ← 적용된 소스에서 표준 테스트 실행 (test-report.md 산출)
/ship            ← commit-only / commit-push / commit-push-pr 범위로 Git 업로드
```

## 작업 유형

| 유형 | 설명 | 산출물 | 아키텍처 계획 | 자동 테스트 |
|------|------|--------|--------------|------------|
| review | 코드를 컨벤션·버그·보안·성능 관점에서 진단. 코드 변경 없이 리포트만 산출. | `work-result.md` (진단 리포트). `changes/` 비어 있음 | 건너뜀 | 건너뜀 |
| bugfix | 버그 원인을 분석하고 수정 패치를 작성. 회귀 방지 테스트도 함께 작성. | `work-result.md` (원인 분석) + `changes/` (수정 패치+테스트) | 폴더 구조 변경 시만 | 실행 |
| generate | 코드베이스 컨벤션과 기능별 폴더 구조에 맞춰 신규 코드를 생성. 테스트 파일도 함께 생성. | `work-result.md` (변경 의도·파일 목록) + `changes/` (신규 파일+테스트) | 항상 실행 (필수) | 실행 |
| refactor | 동작을 보존하면서 코드 구조를 개선. 폴더 재배치가 동반되면 아키텍처 계획 포함. | `work-result.md` (변경 의도·파일 목록) + `changes/` (리팩토링 패치) | 폴더 구조 변경 시만 | 실행 |

## 자동 테스트 동작

- 테스트 명령은 `codebase-context.md`에서 식별된 표준 명령(npm test, pytest, go test ./..., cargo test 등)만 자동 실행한다. 식별되지 않은 임의 명령은 실행하지 않는다.
- generate 유형에서 대상 프로젝트에 테스트 디렉토리·테스트 컨벤션이 없으면 code-generator는 생성 코드에 대응하는 기본 테스트 파일을 `changes/`에 함께 작성한다.
- bug-fixer는 bugfix 시 회귀 방지 테스트를 함께 작성한다.
- 테스트 실패 시 강력 경고를 표시하고, 사용자가 "테스트 실패를 인지하고 업로드"를 명시적으로 승인해야만 Git 업로드로 진행한다.
- 테스트 명령이 식별되지 않거나 런타임이 미설치된 경우 `test-report.md`에 "동적 테스트 미실행" 사유를 명시하고 경고 없이 Git 업로드로 진행한다.

## Git 통합 동작

- Git 업로드는 사용자 최종 승인(체크포인트 B) 이후에만 실행된다.
- Step 7(체크포인트 B)에서 Git 업로드 범위를 선택한다:
  - `commit-only` — 로컬 브랜치 생성 + 커밋만
  - `commit-push` — 커밋 + 원격 push
  - `commit-push-pr` — 커밋 + push + GitHub PR 생성 (gh CLI 필요)
- 변경 사항은 항상 `coding-expert/{session-id}` 브랜치에 커밋된다. 메인 브랜치에 직접 커밋하지 않는다.
- 단계별 폴백: git 레포가 아니면 "git 레포 아님"을 기록하고 종료, remote origin이 없으면 커밋까지만 수행, `gh` 미인증이면 push까지만 수행한다.
