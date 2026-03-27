# oh-my-claude-plugins

Claude Code 플러그인 마켓플레이스입니다.

## 설치

```bash
# 1. 마켓플레이스 등록
/plugin marketplace add kimtaeho-dev/oh-my-claude-plugins

# 2. 플러그인 설치
/plugin install oh-my-project-knowledge@oh-my-claude-plugins

# 3. 플러그인 로드
/reload-plugins
```

로컬 테스트:
```bash
claude --plugin-dir ./plugins/oh-my-claude-plugins
```

## 플러그인 목록

| 플러그인 | 설명 |
|---|---|
| [oh-my-claude-plugins](plugins/oh-my-claude-plugins/) | 코드베이스를 분석하여 프로젝트/비즈니스 지식을 문서화 |

## 레포 구조

```
oh-my-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json               # 마켓플레이스 카탈로그
├── plugins/
│   └── oh-my-claude-plugins/       # 플러그인
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── commands/                   # 슬래시 명령
│       │   ├── init.md
│       │   ├── update.md
│       │   ├── biz-init.md
│       │   ├── biz-update.md
│       │   └── context.md
│       ├── agents/                     # 서브에이전트
│       │   ├── domain-analyzer.md
│       │   └── policy-extractor.md
│       ├── skills/                     # 형식 정의 (자동 참조)
│       │   ├── document-spec/SKILL.md
│       │   └── business-spec/SKILL.md
│       ├── knowledge-system-guide.md
│       └── knowledge-system-examples.md
└── README.md
```

향후 새 플러그인은 `plugins/` 아래에 추가됩니다.
