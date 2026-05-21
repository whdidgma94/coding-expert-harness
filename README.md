# coding-expert-harness

자연어로 요청하면 코드 리뷰·버그 수정·코드 생성·리팩토링을 자동으로 수행하는 Claude Code 하네스입니다.

## 어떻게 동작하나요?

```
사용자 요청 → 유형 분류 → 코드베이스 탐색 → 작업 수행 → 자가 검증 → 사용자 확인 → 실제 적용
```

1. 요청을 4가지 작업 유형(review / bugfix / generate / refactor)으로 분류합니다.
2. 현재 프로젝트의 언어, 프레임워크, 코딩 컨벤션을 자동으로 파악합니다.
3. 유형에 맞는 전문 AI가 작업을 수행합니다.
4. 결과물을 자동으로 검증하고, 문제가 있으면 1회 재시도합니다.
5. 변경 내용을 사용자에게 보여주고 승인을 받은 후에만 실제 파일에 적용합니다.

> 사용자가 승인하기 전까지 실제 소스 파일은 절대 수정되지 않습니다.

## 설치 방법

이 레포지토리를 작업하려는 프로젝트 루트에 복사합니다.

```bash
# 작업할 프로젝트 디렉토리로 이동
cd your-project

# 이 레포지토리의 파일들을 복사
cp -r /path/to/coding-expert-harness/. .
```

또는 Claude Code에서 이 레포를 플러그인으로 등록하여 사용합니다.

## 사용 방법

### 기본 사용 — `/auto`

가장 간단한 방법입니다. 요청만 입력하면 전체 파이프라인이 자동 실행됩니다.

```
/auto "로그인 함수에서 NPE가 발생하는 버그 수정해줘"
/auto "UserService 코드 리뷰해줘"
/auto "결제 모듈에 환불 기능 추가해줘"
/auto "auth/ 디렉토리 리팩토링해줘"
```

### 단계별 사용

전체 파이프라인을 직접 제어하고 싶을 때 사용합니다.

```
/classify "요청"   ← 요청을 작업 유형으로 분류만 수행
/execute           ← 분류 결과 기반으로 작업만 실행
/apply             ← 검증된 변경사항을 실제 파일에 적용
```

## 작업 유형

| 유형 | 설명 | 예시 요청 |
|------|------|-----------|
| **review** | 코드를 진단하고 문제점을 리포트로 정리합니다. 소스는 변경하지 않습니다. | "이 코드 리뷰해줘", "보안 취약점 있는지 확인해줘" |
| **bugfix** | 버그 원인을 분석하고 수정 패치를 만듭니다. | "로그인이 안 돼", "NullPointerException 고쳐줘" |
| **generate** | 새로운 코드를 프로젝트 컨벤션에 맞게 생성합니다. | "회원가입 API 만들어줘", "테스트 코드 작성해줘" |
| **refactor** | 동작은 그대로 유지하면서 코드 구조를 개선합니다. | "중복 코드 제거해줘", "함수가 너무 길어서 분리해줘" |

## 산출물

작업이 실행되면 `.coding-expert-artifacts/{세션ID}/` 폴더에 다음 파일들이 생성됩니다.

```
.coding-expert-artifacts/
└── 20260521-fix-login-npe/
    ├── request.md          ← 사용자 요청 원문
    ├── task-spec.md        ← 작업 분류 및 완료 기준
    ├── codebase-context.md ← 탐색한 코드베이스 정보
    ├── work-result.md      ← 작업 결과 요약
    ├── changes/            ← 실제 적용될 변경 파일들
    ├── verification.md     ← 자가 검증 결과
    └── apply-log.md        ← 적용 결과 로그
```

## 내부 구조

이 하네스는 역할이 분리된 여러 AI 에이전트로 구성됩니다.

| 에이전트 | 역할 | 모델 |
|---------|------|------|
| intent-classifier | 요청을 4가지 유형으로 분류 | Opus |
| codebase-explorer | 프로젝트 구조·컨벤션 탐색 | Sonnet |
| code-reviewer | 코드 진단 리포트 작성 | Opus |
| bug-fixer | 버그 원인 분석 및 패치 생성 | Sonnet |
| code-generator | 신규 코드 생성 | Sonnet |
| refactorer | 리팩토링 패치 생성 | Sonnet |
| work-verifier | 결과물 품질 검증 (PASS/REVISE) | Opus |
| change-applier | 승인된 패치를 실제 파일에 적용 | Haiku |
| git-operator | 브랜치 생성·커밋·push·PR 생성 | Haiku |

## git 작업 옵션

변경 사항 적용 시 git 작업 범위를 선택할 수 있습니다.

| 옵션 | 동작 |
|------|------|
| `commit-only` | 로컬에 새 브랜치 생성 + 커밋 |
| `commit-push` | 커밋 + 원격 push |
| `commit-push-pr` | 커밋 + push + GitHub PR 생성 (`gh` CLI 필요) |

변경 사항은 항상 `coding-expert/{세션ID}` 브랜치에 커밋됩니다. 메인 브랜치에 직접 커밋하지 않습니다.
