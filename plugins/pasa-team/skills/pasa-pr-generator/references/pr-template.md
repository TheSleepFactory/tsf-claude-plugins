# PR Template Reference

이 파일은 Pasa PR Generator가 PR 본문을 생성할 때 참조하는 템플릿입니다.

## 기본 템플릿 구조

```markdown
## Summary
- [주요 변경사항 1]
- [주요 변경사항 2]
- [주요 변경사항 3]

## Changes
### ✨ Features
- scope: 설명

### 🐛 Bug Fixes
- scope: 설명

### ♻️ Refactoring
- scope: 설명

### 📝 Documentation
- scope: 설명

### 🧪 Tests
- scope: 설명

## Test plan
- [ ] 테스트 항목 1
- [ ] 테스트 항목 2
- [ ] 테스트 항목 3

---
🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

## 섹션별 가이드라인

### Summary
- 3-5개의 bullet point로 구성
- 사용자 관점에서 "무엇이 바뀌었는가"를 설명
- 기술적 세부사항보다 비즈니스 가치 강조

### Changes (커밋 5개 이상 시 표시)
- Conventional Commits 타입별로 그룹화
- 각 카테고리는 해당 커밋이 있을 때만 표시
- scope가 있으면 `scope: 설명` 형식, 없으면 `설명`만

### Test plan
- 변경사항과 관련된 테스트 시나리오
- 체크박스 형식으로 리뷰어가 확인 가능하도록
- 최소 2개, 최대 5개 항목

### Footer
- Claude Code 생성 표시 필수 포함

## 커밋 타입 이모지 매핑

| Type | Emoji | Label |
|------|-------|-------|
| feat | ✨ | enhancement |
| fix | 🐛 | bug |
| docs | 📝 | documentation |
| refactor | ♻️ | - |
| test | 🧪 | - |
| chore | 🔧 | - |
| style | 💄 | - |
| perf | ⚡ | performance |
| ci | 👷 | - |
| build | 📦 | - |
