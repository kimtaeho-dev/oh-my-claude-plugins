# oh-my-claude-plugins

Claude Code 플러그인 마켓플레이스입니다.

## 설치

```bash
# 1. 마켓플레이스 등록
/plugin marketplace add kimtaeho-dev/oh-my-claude-plugins

# 2. 플러그인 설치
/plugin install oh-my-project-knowledge@oh-my-claude-plugins
/plugin install oh-my-claude-setup@oh-my-claude-plugins

# 3. 플러그인 로드
/reload-plugins
```

로컬 테스트:
```bash
claude --plugin-dir ./plugins/oh-my-claude-plugins
```

## 플러그인 목록

| 플러그인 | 버전 | 설명 |
|---|---|---|
| [oh-my-project-knowledge](plugins/oh-my-project-knowledge/) | 0.2.0 | 코드베이스를 분석하여 프로젝트/비즈니스 지식을 문서화 |
| [oh-my-claude-setup](plugins/oh-my-claude-setup/) | 0.2.0 | 프로젝트에 이상적인 Claude Code 초기 환경을 자동 구성 |

### oh-my-project-knowledge

프로젝트 코드베이스를 분석하여 도메인 구조, 영향 관계, 비즈니스 규칙, 용어집, 기획서를 자동 생성합니다.

| 명령 | 설명 |
|---|---|
| `/oh-my-project-knowledge:init` | 프로젝트 지식 문서 초기 생성 |
| `/oh-my-project-knowledge:update` | 코드 변경 시 문서 갱신 |
| `/oh-my-project-knowledge:biz-init` | 비즈니스 지식 문서 초기 생성 |
| `/oh-my-project-knowledge:biz-update` | 비즈니스 지식 문서 갱신 |
| `/oh-my-project-knowledge:context` | 개발 전 컨텍스트 로딩 |

### oh-my-claude-setup

프로젝트에 이상적인 Claude Code 초기 환경(CLAUDE.md, 원칙 파일, 이슈 트래킹)을 자동 구성합니다. 단일 앱 / 모노레포 / MSA 프로젝트 유형을 자동 판별하여 대응합니다.

| 명령 | 설명 |
|---|---|
| `/oh-my-claude-setup:init` | Claude Code 초기 환경 구성 |

생성되는 파일:
- `CLAUDE.md` — 프로젝트 유형별 구조화된 프로젝트 안내서
- `.claude/rules/principles/` — 작업 시작, 코드 수정, Git 워크플로우 등 원칙 파일 7종
- `.claude/tasks/known-issues.md` — TODO/FIXME 수집
- `.claude/tasks/suspected-bugs.md` — 버그 의심 패턴 트래킹

## 레포 구조

```
oh-my-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json               # 마켓플레이스 카탈로그
├── plugins/
│   ├── oh-my-project-knowledge/       # 프로젝트 지식 문서화 플러그인
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   ├── commands/                   # 슬래시 명령
│   │   │   ├── init.md
│   │   │   ├── update.md
│   │   │   ├── biz-init.md
│   │   │   ├── biz-update.md
│   │   │   └── context.md
│   │   ├── agents/                     # 서브에이전트
│   │   │   ├── domain-analyzer.md
│   │   │   └── policy-extractor.md
│   │   ├── skills/                     # 형식 정의 (자동 참조)
│   │   │   ├── document-spec/SKILL.md
│   │   │   ├── business-spec/SKILL.md
│   │   │   └── website-spec/SKILL.md
│   │   ├── knowledge-system-guide.md
│   │   └── knowledge-system-examples.md
│   └── oh-my-claude-setup/            # Claude Code 초기 환경 구성 플러그인
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── commands/
│       │   └── init.md
│       └── skills/
│           ├── claude-md-spec/SKILL.md
│           └── principles-spec/SKILL.md
└── README.md
```

향후 새 플러그인은 `plugins/` 아래에 추가됩니다.
