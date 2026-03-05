---
name: pasa-pr-generator
description: This skill analyzes commits between current branch and develop, then creates a well-structured PR with auto-generated title, summary, test plan, and labels. Triggers on "/pasa-pr-generator" command.
---

# Pasa PR Generator

현재 브랜치의 develop 대비 커밋을 분석하여 구조화된 PR을 자동 생성합니다.

## 워크플로우

이 스킬이 트리거되면 다음 단계를 **순서대로** 실행하세요:

### 1단계: 사전 검증

다음 명령어들을 **병렬로** 실행하여 현재 상태를 확인합니다:

```bash
# 현재 브랜치 이름 확인
git branch --show-current

# develop 대비 새 커밋 확인
git log develop..HEAD --oneline

# 원격 브랜치 존재 확인
git ls-remote --heads origin $(git branch --show-current)

# unstaged 변경사항 확인
git status --porcelain

# 이미 PR이 존재하는지 확인
gh pr list --head $(git branch --show-current) --state open --json url,number
```

#### 검증 실패 시 처리

| 상황 | 처리 방법 |
|------|-----------|
| 현재 브랜치가 `develop`인 경우 | "❌ develop 브랜치에서는 PR을 생성할 수 없습니다. feature 브랜치로 이동해주세요." 메시지 출력 후 **중단** |
| 새 커밋이 없는 경우 | "❌ develop 대비 새로운 커밋이 없습니다. 먼저 커밋을 추가해주세요." 메시지 출력 후 **중단** |
| 원격 브랜치가 없는 경우 | "⚠️ 원격 브랜치가 없습니다. `git push -u origin [브랜치명]`을 먼저 실행해주세요." 메시지 출력 후 **중단** |
| unstaged 변경사항이 있는 경우 | "⚠️ 커밋되지 않은 변경사항이 있습니다. PR에 포함하시겠습니까?" 라고 사용자에게 확인 |
| 이미 열린 PR이 존재하는 경우 | "ℹ️ 이미 열린 PR이 있습니다: [PR URL]. 기존 PR을 업데이트하시겠습니까?" 라고 사용자에게 확인 |

### 2단계: 커밋 분석

develop 대비 모든 커밋을 수집하고 분석합니다:

```bash
# 상세 커밋 로그 수집 (해시, 제목, 본문)
git log develop..HEAD --pretty=format:"%h|%s|%b---COMMIT_END---"
```

#### Conventional Commits 파싱

각 커밋 메시지를 파싱하여 다음 정보를 추출합니다:
- **type**: feat, fix, refactor, docs, test, chore, style, perf, ci, build
- **scope**: 괄호 안의 범위 (선택적)
- **description**: 콜론 뒤의 설명

파싱 패턴: `^(feat|fix|refactor|docs|test|chore|style|perf|ci|build)(\([^)]+\))?:\s*(.+)$`

**Non-conventional 커밋 처리**: 위 패턴에 맞지 않는 커밋은 `type: chore`, `scope: null`, `description: 전체 메시지`로 처리합니다.

#### 타입별 그룹화

파싱된 커밋들을 타입별로 그룹화합니다:
```
{
  feat: [{scope, description}, ...],
  fix: [{scope, description}, ...],
  refactor: [{scope, description}, ...],
  ...
}
```

### 3단계: PR 타이틀 생성

#### 브랜치명에서 대주제 추출

브랜치명 패턴 분석:
- `feature/app-514` → 대주제: `app-514`
- `fix/login-bug` → 대주제: `login-bug`
- `refactor/auth-module` → 대주제: `auth-module`

추출 규칙: 첫 번째 `/` 이후의 부분을 대주제로 사용

#### 핵심 기능 요약

1. `feat` 타입 커밋이 있으면 가장 중요한 feat 커밋의 description 사용
2. `feat`가 없으면 가장 많은 커밋이 있는 타입에서 대표 description 선택
3. 여러 feat 커밋이 있으면 공통 주제로 요약

#### 타이틀 형식

```
{type}({대주제}): {핵심 기능 요약}
```

- 최대 **72자** 제한
- type은 가장 많은 커밋 타입 또는 feat (feat 우선)
- 예시: `feat(app-514): 지도 컴포넌트 Mapbox Access Token 초기화 중앙화`

### 4단계: PR 본문 생성

`references/pr-template.md`를 참조하여 본문을 생성합니다.

#### Summary 섹션

커밋 분석 결과를 바탕으로 **3-5개의 bullet point**로 요약합니다:
- 사용자 관점에서 "무엇이 바뀌었는가"를 설명
- 기술적 세부사항보다 비즈니스 가치를 강조
- 각 bullet point는 한 문장으로 간결하게

예시:
```markdown
## Summary
- Mapbox Access Token 초기화를 App.tsx로 중앙화하여 일관성 향상
- 지도 컴포넌트의 중복 코드 제거로 유지보수성 개선
- 피드백 모달의 X 버튼 동작 수정
```

#### Changes 섹션 (조건부)

**커밋이 5개 이상**인 경우에만 이 섹션을 추가합니다.

타입별로 커밋을 그룹화하여 상세 목록을 작성합니다:

```markdown
## Changes
### ✨ Features
- mapbox: 지도 컴포넌트 Access Token 설정 제거
- app: Mapbox Access Token 초기화를 App.tsx로 중앙화

### 🐛 Bug Fixes
- feedback-modal: X 버튼 클릭 시 피드백 완료 처리

### ♻️ Refactoring
- dental-clinic-map: 코드 스타일 정리 및 헤더 높이 상수화
```

각 타입의 이모지 매핑:
- feat → ✨
- fix → 🐛
- refactor → ♻️
- docs → 📝
- test → 🧪
- chore → 🔧
- style → 💄
- perf → ⚡
- ci → 👷
- build → 📦

해당 타입의 커밋이 없으면 그 섹션은 생략합니다.

#### Test plan 섹션

변경사항에 기반하여 **2-5개의 테스트 체크리스트**를 생성합니다:

```markdown
## Test plan
- [ ] 앱 시작 시 Mapbox 지도가 정상적으로 로드되는지 확인
- [ ] 치과 클리닉 지도 화면에서 마커가 올바르게 표시되는지 확인
- [ ] 피드백 모달의 X 버튼 클릭 시 올바르게 닫히는지 확인
```

테스트 항목 생성 규칙:
1. 각 `feat` 커밋에 대해 기능 테스트 항목 생성
2. 각 `fix` 커밋에 대해 버그 수정 확인 항목 생성
3. `refactor` 커밋에 대해 기존 기능 동작 확인 항목 생성

#### Footer

반드시 다음 footer를 포함합니다:

```markdown
---
🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### 5단계: 라벨 결정

`config/label-mapping.json`을 참조하여 라벨을 결정합니다.

#### 라벨 매핑 규칙

1. 커밋 타입별 라벨 매핑을 확인합니다
2. `null`로 매핑된 타입은 라벨을 추가하지 않습니다
3. 여러 타입이 있으면 해당하는 **모든 라벨**을 추가합니다

예시:
- feat 커밋이 있으면 → `enhancement` 라벨
- fix 커밋이 있으면 → `bug` 라벨
- feat + fix 둘 다 있으면 → `enhancement`, `bug` 두 라벨 모두

#### 라벨 존재 확인

```bash
# 저장소의 라벨 목록 확인
gh label list --json name
```

저장소에 존재하지 않는 라벨은 **추가하지 않습니다**.

### 6단계: PR 생성

모든 정보가 준비되면 PR을 생성합니다:

```bash
gh pr create \
  --title "PR 타이틀" \
  --body "$(cat <<'EOF'
## Summary
- ...

## Changes (있는 경우)
...

## Test plan
- [ ] ...

---
🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)" \
  --base develop \
  --label "라벨1,라벨2"
```

#### 주의사항

- `--base`는 항상 `develop`
- `--label`은 존재하는 라벨만 포함 (없으면 옵션 생략)
- `--reviewer`는 지정하지 않음
- 본문에 특수문자가 있을 수 있으므로 HEREDOC 사용

### 7단계: 완료 보고

PR 생성이 성공하면 다음 정보를 사용자에게 보고합니다:

```
✅ PR이 성공적으로 생성되었습니다!

📋 PR 정보
- 타이틀: {타이틀}
- URL: {PR URL}
- 라벨: {라벨 목록}
- Base: develop
- 커밋 수: {커밋 개수}
```

---

## 전체 프로세스 요약

```
1. 사전 검증 (브랜치, 커밋, 원격, unstaged, 기존 PR)
     ↓
2. 커밋 분석 (Conventional Commits 파싱)
     ↓
3. PR 타이틀 생성 (72자 제한)
     ↓
4. PR 본문 생성 (Summary, Changes, Test plan, Footer)
     ↓
5. 라벨 결정 (타입 → 라벨 매핑)
     ↓
6. PR 생성 (gh pr create)
     ↓
7. 완료 보고
```

## 에러 처리

| 에러 상황 | 처리 방법 |
|----------|-----------|
| gh CLI 미설치 | "❌ GitHub CLI(gh)가 설치되어 있지 않습니다. brew install gh로 설치해주세요." |
| gh 인증 안됨 | "❌ GitHub CLI 인증이 필요합니다. gh auth login을 실행해주세요." |
| 네트워크 에러 | "❌ GitHub에 연결할 수 없습니다. 네트워크 연결을 확인해주세요." |
| 권한 부족 | "❌ 이 저장소에 PR을 생성할 권한이 없습니다." |

## 참조 파일

- `references/pr-template.md`: PR 본문 템플릿 및 가이드라인
- `config/label-mapping.json`: 커밋 타입 → 라벨 매핑 설정
