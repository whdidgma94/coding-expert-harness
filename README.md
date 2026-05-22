# coding-expert-harness

자연어로 요청하면 기능별 폴더 구조 설계부터 코드 생성·자동 테스트·Git 업로드까지 전체 개발 작업을 자동으로 수행하는 Claude Code 하네스입니다.

## 주요 기능

- **기능별 폴더 구조 설계** — 코드를 생성할 때 관련 기능을 같은 폴더에 모으는 feature 단위 계층 구조를 먼저 설계하고 사용자와 확정합니다. 기존 코드베이스 패턴을 우선 따릅니다.
- **사전 계획 후 실행** — 코드를 생성하기 전에 폴더 트리·모듈 경계·파일 경로를 사용자와 함께 확정합니다. 구조가 확정된 뒤에야 코드를 생성합니다.
- **자동 테스트** — 변경 사항이 실제 소스에 적용된 후 표준 테스트 명령(npm test, pytest, go test 등)을 자동으로 실행합니다. 테스트 파일이 없는 프로젝트에서는 테스트 코드도 함께 생성합니다.
- **Git 통합** — 사용자 최종 승인 후 브랜치 생성→커밋→push→PR 생성을 자동으로 수행합니다. commit-only / commit-push / commit-push-pr 중 원하는 범위를 선택할 수 있습니다.

## 빠른 시작

```
# 새 기능 추가 (폴더 구조 설계 → 코드 생성 → 테스트 → Git 업로드)
/auto "결제 모듈에 환불 기능 추가해줘"

# 버그 수정 (원인 분석 → 패치 생성 → 회귀 테스트 → 적용)
/auto "로그인 함수에서 NPE가 발생하는 버그 수정해줘"
```

## 파이프라인 흐름

```
/auto "요청" 실행 시

[Step 1] 요청 접수 & 세션 생성
    ↓
[Step 2] 의도 분류 (review / bugfix / generate / refactor)
    ↓
[Step 3] 코드베이스 탐색 (언어·프레임워크·폴더 구조·테스트 명령 파악)
    ↓
[Step 4] 아키텍처 계획 (기능별 폴더 트리·파일 배치 설계)
    ↓
    *** 사용자 확인 체크포인트 A: 폴더 구조 승인 ***
    ↓
[Step 5] 작업 수행 (유형별 전문 AI 분기 실행)
    ↓
[Step 6] 자가 검증 (PASS / REVISE → 최대 1회 재시도)
    ↓
    *** 사용자 확인 체크포인트 B: diff 확인 + Git 범위 선택 ***
    ↓
[Step 8] 변경 적용 (changes/ → 실제 소스)
    ↓
[Step 9] 자동 테스트 (표준 테스트 명령 실행)
    ↓
[Step 10] Git 업로드 (브랜치 생성→커밋→push→PR)

* review 유형은 체크포인트 B에서 리포트 전달 후 종료 (Step 8~10 건너뜀)
* 아키텍처 계획(Step 4)은 generate에서 필수, refactor·bugfix는 폴더 변경 시만, review는 건너뜀
```

## 커맨드 목록

| 커맨드 | 설명 |
|--------|------|
| `/auto "요청"` | 전체 파이프라인 실행 (분류→탐색→설계→수행→검증→적용→테스트→Git) |
| `/classify "요청"` | 요청을 작업 유형으로 분류만 수행 (`task-spec.md` 산출) |
| `/design` | 승인된 `task-spec.md` 기반으로 기능별 폴더 구조 설계만 실행 (`architecture-plan.md` 산출) |
| `/execute` | 승인된 `architecture-plan.md` 기반으로 작업 수행만 실행 |
| `/apply` | 검증·승인된 `changes/`를 실제 소스에 적용 |
| `/test` | 적용된 소스에서 표준 테스트 실행 (`test-report.md` 산출) |
| `/ship` | `commit-only` / `commit-push` / `commit-push-pr` 범위로 Git 업로드 |

## 작업 유형

| 유형 | 설명 | 아키텍처 계획 | 자동 테스트 | 예시 요청 |
|------|------|:------------:|:-----------:|-----------|
| **review** | 코드를 진단하고 문제점을 리포트로 정리. 소스 변경 없음. | 건너뜀 | 건너뜀 | "이 코드 리뷰해줘", "보안 취약점 확인해줘" |
| **bugfix** | 버그 원인 분석 + 수정 패치 + 회귀 방지 테스트 생성 | 폴더 변경 시만 | 실행 | "로그인이 안 돼", "NPE 고쳐줘" |
| **generate** | 기능별 폴더 구조에 맞춰 신규 코드·테스트 파일 생성 | 항상 실행 | 실행 | "환불 API 만들어줘", "회원가입 기능 추가해줘" |
| **refactor** | 동작 보존 리팩토링. 폴더 재배치 시 아키텍처 계획 포함. | 폴더 변경 시만 | 실행 | "중복 코드 제거해줘", "함수 분리해줘" |

## 산출물 구조

작업이 실행되면 `.coding-expert-artifacts/{세션ID}/` 폴더에 다음 파일들이 생성됩니다.

```
.coding-expert-artifacts/
└── 20260521-add-refund-feature/
    ├── request.md           ← 사용자 요청 원문
    ├── task-spec.md         ← 작업 분류 및 완료 기준
    ├── codebase-context.md  ← 탐색한 코드베이스 정보 (언어·구조·테스트 명령)
    ├── architecture-plan.md ← 기능별 폴더 트리·파일 배치 (체크포인트 A 승인본)
    ├── work-result.md       ← 작업 결과 요약
    ├── changes/             ← 실제 적용될 변경 파일들 (폴더 구조 미러링)
    │   └── src/features/payment/refund.ts
    │   └── src/features/payment/refund.test.ts
    ├── verification.md      ← 자가 검증 결과 (PASS/REVISE)
    ├── test-report.md       ← 자동 테스트 실행 결과
    └── apply-log.md         ← 적용 결과 + git 작업 결과 (브랜치·커밋·PR URL)
```

## 내부 에이전트 구성

| 에이전트 | 역할 | 모델 |
|---------|------|------|
| intent-classifier | 요청을 4가지 유형으로 분류 | Opus |
| codebase-explorer | 프로젝트 구조·컨벤션·테스트 명령 탐색 | Sonnet |
| architecture-planner | 기능별 폴더 구조·모듈 경계·파일 배치 설계 | Opus |
| code-reviewer | 코드 진단 리포트 작성 | Opus |
| bug-fixer | 버그 원인 분석 + 패치 + 회귀 테스트 생성 | Sonnet |
| code-generator | 기능별 폴더 구조에 맞춘 신규 코드 + 테스트 생성 | Sonnet |
| refactorer | 동작 보존 리팩토링 패치 생성 | Sonnet |
| work-verifier | 결과물 품질 검증 (PASS/REVISE) | Opus |
| change-applier | 승인된 패치를 실제 파일에 적용 | Haiku |
| git-operator | 브랜치 생성·커밋·push·PR 생성 | Haiku |
| test-runner | 표준 테스트 명령 실행 및 결과 기록 | Sonnet |

## Git 업로드 옵션

체크포인트 B에서 다음 중 하나를 선택합니다.

| 옵션 | 동작 |
|------|------|
| `commit-only` | 로컬에 새 브랜치 생성 + 커밋 |
| `commit-push` | 커밋 + 원격 push |
| `commit-push-pr` | 커밋 + push + GitHub PR 생성 (`gh` CLI 필요) |

변경 사항은 항상 `coding-expert/{세션ID}` 브랜치에 커밋됩니다. 메인 브랜치에 직접 커밋하지 않습니다.
