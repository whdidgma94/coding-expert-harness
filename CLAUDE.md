# coding-expert-harness

자연어 요청을 받아 코드 리뷰·버그 수정·코드 생성·리팩토링을 자동 수행하는 범용 코딩 보조 하네스.

## 산출물 경로

모든 작업 산출물은 `.coding-expert-artifacts/{session-id}/` 하위에 저장한다.
`{session-id}`는 요청 접수 시각과 작업 키워드 기반으로 결정한다 (예: `20260521-fix-login-npe`). 소문자 + 하이픈 + 숫자만 사용한다.

산출물 목록:
- `request.md` — 사용자의 원문 자연어 요청 보존 (Step 1)
- `task-spec.md` — 분류된 작업 유형(review/bugfix/generate/refactor), 작업 범위, 대상 파일 후보, 요청 충족 기준 (Step 2)
- `codebase-context.md` — 대상 코드베이스의 언어·프레임워크·디렉토리 구조·코딩 컨벤션·테스트/빌드 명령 (Step 3)
- `work-result.md` — 작업 수행 결과 요약. review는 진단 리포트, bugfix는 원인 분석, generate/refactor는 변경 의도와 파일 목록 (Step 4)
- `changes/` — 실제 소스에 적용될 변경 단위 모음. 파일별 신규 내용 또는 unified diff 패치. review 유형에서는 비어 있다 (Step 4)
- `verification.md` — 자가 검증 결과(PASS/REVISE), 체크리스트, REVISE 시 재실행 지시사항 (Step 5)
- `apply-log.md` — 실제 소스에 적용된 파일 목록과 적용 결과 (Step 7)

## 파이프라인 규칙

1. **소스 쓰기 범위 제한**: Step 1~6의 모든 agent는 `.coding-expert-artifacts/{session-id}/` 산출물 디렉토리에만 쓰기를 수행한다. 실제 소스 파일은 절대 수정하지 않는다. 실제 소스 수정은 Step 7의 `change-applier` agent만 수행하며, git 작업(브랜치·커밋·push·PR)은 `git-operator` agent가 담당한다. 두 agent 모두 사용자 승인 이후에만 실행된다.
2. **읽기 권한**: codebase-explorer, worker agents, work-verifier는 대상 코드베이스 전체에 대한 읽기(Read/Glob/Grep)와 검증용 명령 실행(Bash: 테스트·빌드·린트)이 허용된다. Bash는 읽기·검증 목적에 한정하며 소스 파일을 변경하는 명령은 금지한다.
3. **분기 실행**: Step 4는 `task-spec.md`의 작업 유형에 따라 worker agent 1개만 실행한다. 4개 유형 worker를 병렬 실행하지 않는다.
4. **검증 루프**: Step 5에서 work-verifier가 REVISE 판정 시 verification.md의 지시사항을 입력으로 Step 4를 1회 재실행한다. 재실행 후에도 REVISE면 검증 결과를 사용자에게 보여주고 진행 여부를 묻는다 (최대 1회 재시도).
5. **사용자 확인 체크포인트**: Step 6에서 사용자 승인 없이는 Step 7(실제 소스 적용)로 진행하지 않는다. 체크포인트에서 작업 요약 + 진단 결과 + 적용될 diff를 함께 제시한다. 사용자가 수정을 요청하면 해당 부분만 Step 4로 돌아가 재작업한다.
6. **review 유형 종료 처리**: 작업 유형이 review인 경우 `changes/`가 비어 있으므로 Step 7을 건너뛰고, Step 6에서 진단 리포트를 사용자에게 전달하는 것으로 파이프라인을 종료한다.
7. **변경 적용 방식**: `change-applier`가 파일을 적용한 뒤 `git-operator`가 git 작업을 수행한다. git 레포이면 새 브랜치(`coding-expert/{session-id}`)를 생성하고 커밋한다. push 및 PR 생성 여부는 Step 7 체크포인트에서 사용자가 선택한다. git이 없는 코드베이스에서는 직접 파일을 수정하고 git-operator는 실행하지 않는다.
8. **코드베이스 위치**: 하네스가 실행되는 현재 디렉토리(cwd)를 항상 작업 대상으로 사용한다.
9. **테스트 실행 범위**: `codebase-context.md`에서 식별된 표준 명령(npm test, pytest 등)만 자동 실행한다. 식별되지 않은 명령은 실행하지 않는다.
10. **신규 파일 경로 확정**: generate 유형에서 사용자가 경로를 지정하지 않으면 code-generator가 경로를 제안하고 Step 6 체크포인트에서 사용자 확인 후 확정한다.
11. **외부 의존성 처리**: 테스트·빌드·린트 실행에 필요한 도구가 미설치된 경우, work-verifier는 "정적 검토만 수행함, 동적 검증 미실행"을 verification.md에 명시하고 PASS/REVISE 판정을 정적 근거로 내린다.
12. **세션 격리**: 각 요청은 고유 `{session-id}` 디렉토리를 가지며, 이전 세션의 산출물을 참조하거나 덮어쓰지 않는다.

## 커맨드

```
/auto "요청"        ← 자연어 요청을 받아 분류→탐색→수행→검증→적용 전체 파이프라인 실행
/classify "요청"    ← 요청을 작업 유형으로 분류만 수행 (task-spec.md 산출)
/execute            ← 승인된 task-spec.md 기반으로 작업 수행만 실행
/apply              ← 검증·승인된 changes/ 를 실제 소스에 적용 (파일 적용 + git 작업)
```

git 작업 범위는 `/apply` 실행 시 선택한다:
- `commit-only` — 로컬 브랜치 생성 + 커밋만
- `commit-push` — 커밋 + 원격 push
- `commit-push-pr` — 커밋 + push + GitHub PR 생성 (gh CLI 필요)

## 작업 유형

| 유형 | 설명 | 산출물 |
|------|------|--------|
| review | 코드를 컨벤션·버그·보안·성능 관점에서 진단 | `work-result.md` (진단 리포트). `changes/` 비어 있음. 코드 변경 없음. |
| bugfix | 버그 원인을 분석하고 수정 패치를 작성 | `work-result.md` (원인 분석) + `changes/` (수정 패치) |
| generate | 코드베이스 컨벤션에 맞는 신규 코드를 생성 | `work-result.md` (변경 의도·파일 목록) + `changes/` (신규 파일) |
| refactor | 동작을 보존하면서 코드 구조를 개선 | `work-result.md` (변경 의도·파일 목록) + `changes/` (리팩토링 패치) |
