# Pasa Claude Plugins

Pasa 팀 공용 Claude Code 플러그인 마켓플레이스입니다.

## 구조

```
pasa-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json       ← 마켓플레이스 정의
└── plugins/
    └── pasa-team/             ← 팀 공용 플러그인
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            ├── pasa-commit-pusher/
            └── pasa-pr-generator/
```

## 사용법

프로젝트의 `.claude/settings.json`에서 마켓플레이스를 등록하면 자동으로 플러그인을 사용할 수 있습니다.

자세한 설치 방법은 [pasa-team README](./plugins/pasa-team/README.md)를 참조하세요.
