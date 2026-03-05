# Pasa Commit Pusher

현재 unstaged된 변경사항을 분석하고 논리적 단위로 나누어 커밋한 후 원격 저장소에 푸시합니다.

## 작업 내용

1. **변경사항 분석**: `git status`와 `git diff`를 통해 모든 unstaged 변경사항을 확인합니다.
2. **논리적 그룹화**: 변경된 파일들을 기능, 목적, 또는 관련성에 따라 논리적 단위로 그룹화합니다.
3. **커밋 생성**: 각 그룹에 대해 의미 있는 커밋 메시지(한국어)를 작성하고 커밋합니다.
4. **원격 푸시**: 모든 커밋이 완료되면 원격 저장소에 푸시합니다.

## 커밋 메시지 규칙

- **한국어**로 작성
- **Conventional Commits** 형식 준수: `type(scope): 설명`
- 타입: `feat`, `fix`, `refactor`, `style`, `docs`, `chore`, `test` 등

## 실행

pasa-commit-pusher 에이전트를 사용하여 위 작업을 수행해주세요.
