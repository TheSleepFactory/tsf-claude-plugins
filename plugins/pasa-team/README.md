# Pasa Team Plugin

Pasa 팀 공용 Claude Code 개발 워크플로우 스킬 모음입니다.

## 스킬 목록

### pasa-commit-pusher
unstaged 변경사항을 분석하여 논리적 단위로 커밋하고 원격 저장소에 푸시합니다.

**호출**: `/pasa-team:pasa-commit-pusher`

### pasa-pr-generator
현재 브랜치의 커밋을 분석하여 구조화된 PR을 자동 생성합니다.

**호출**: `/pasa-team:pasa-pr-generator`

## 설치

각 프로젝트의 `.claude/settings.json`에 다음을 추가:

```json
{
  "extraKnownMarketplaces": {
    "pasa-plugins": {
      "source": {
        "source": "github",
        "repo": "TheSleepFactory/pasa-claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "pasa-team@pasa-plugins": true
  }
}
```
