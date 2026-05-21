---
name: explore
description: Step 3 담당. auto skill이 Step 3에서 호출할 때 실행된다. 대상 코드베이스(cwd)를 탐색하여 언어·프레임워크·디렉토리 구조·코딩 컨벤션·테스트 방식을 파악하고 codebase-context.md를 산출한다.
---

# explore skill

현재 작업 디렉토리(cwd)의 코드베이스를 탐색하여 코딩 컨벤션, 구조, 테스트 방식을 파악하고 후속 단계(execute, verify)가 활용할 `codebase-context.md`를 생성한다.

## 입력

- `{session-id}`: 현재 세션 식별자
- `.coding-expert-artifacts/{session-id}/task-spec.md`: 대상 파일 후보 및 작업 유형

---

## Step 1 — codebase-explorer agent 호출

**sub-agent를 호출**하여 codebase-explorer agent를 실행한다.

agent에게 전달하는 인자:
- `session-id`: {실제 session-id 값}

agent는 `.coding-expert-artifacts/{session-id}/task-spec.md`와 cwd를 직접 탐색하여 나머지 컨텍스트를 확보한다.

**출력**: `.coding-expert-artifacts/{session-id}/codebase-context.md`

---

## Step 2 — codebase-context.md 구조 검증

생성된 `codebase-context.md`가 다음 항목을 포함하는지 확인한다:

- `언어 및 프레임워크`: 주요 언어, 프레임워크, 핵심 의존성
- `디렉토리 구조`: 소스 루트, 테스트 디렉토리, 설정 파일 위치 요약
- `코딩 컨벤션`: 네이밍 컨벤션, 임포트 스타일, 린터/포매터 설정 요약
- `테스트 방식`: 테스트 프레임워크, 테스트 파일 위치 패턴
- `표준 실행 명령`: 테스트·빌드·린트 명령 목록 (verify skill이 사용)
- `git 여부`: `true` 또는 `false`, git인 경우 현재 브랜치

항목이 누락된 경우 codebase-explorer agent를 다시 호출하여 보완한다.

---

## 출력

- `.coding-expert-artifacts/{session-id}/codebase-context.md` — 코드베이스 컨텍스트 전체
