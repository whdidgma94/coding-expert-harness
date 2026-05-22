# explore

Step 3 담당. 대상 코드베이스(cwd)를 탐색하여 언어·프레임워크·기존 디렉토리 구조·기능별 폴더 패턴·코딩 컨벤션·표준 테스트/빌드 명령·테스트 디렉토리 존재 여부를 파악하고 `codebase-context.md`를 산출한다.

## 입력

- `session-id`: 현재 세션 식별자

## 실행 절차

`codebase-explorer` agent를 호출한다.
- 전달 인자: `session-id`

agent는 `.coding-expert-artifacts/{session-id}/task-spec.md`와 cwd를 직접 탐색하여 나머지 컨텍스트를 확보하고, `codebase-context.md`를 직접 작성한다.

## 출력

- `.coding-expert-artifacts/{session-id}/codebase-context.md`
