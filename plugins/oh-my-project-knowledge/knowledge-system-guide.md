# 프로젝트 지식 체계 가이드

---

## 이 문서는 무엇인가

이 문서는 Claude Code가 코드베이스를 분석하여 **프로젝트 지식**과 **비즈니스 지식**을 체계적으로 문서화하는 시스템의 전체 그림을 설명한다.

설정/실행 방법은 각 Skill에, 산출물 예시는 `knowledge-system-examples.md`에 별도로 정리되어 있다.

---

## 해결하고자 하는 문제

### 문제 1: 영향 범위를 모른다

프로젝트 규모가 커지면 "이 기능을 수정하면 어디에 영향이 가는가?"를 파악하기 어렵다. 코드베이스를 처음 보는 사람은 물론, 오래 참여한 개발자도 도메인 간 의존 관계를 놓치고 사이드 이펙트를 만든다.

### 문제 2: 비즈니스 규칙이 코드에 묻혀 있다

validation 조건, 에러 메시지, 상수, enum, 상태 전이 로직 — 이것들이 곧 서비스 정책이지만, 코드를 직접 읽지 않으면 알 수 없다. 기획자가 "최소 참가자 수가 몇 명이지?"라고 물으면 개발자가 코드를 뒤져야 하고, 신규 팀원이 "이 에러는 언제 발생하지?"를 파악하려면 소스 코드를 추적해야 한다.

---

## 해결 방법: 두 단계 파이프라인

```
┌─────────────────────────────────┐
│  1단계: 코드 지식 체계            │
│  "어디에 영향이 가는가?" (구조)    │
│                                 │
│  코드베이스 분석 →                │
│  DOMAIN_MAP / FEATURES /         │
│  ENTITIES / SERVICE_FLOW /       │
│  DEPENDENCY_MATRIX              │
└──────────┬──────────────────────┘
           │ 입력 (기능 ID가 연결 고리)
           ▼
┌─────────────────────────────────┐
│  2단계: 비즈니스 지식 체계         │
│  "시스템이 어떻게 동작하는가?" (규칙)│
│                                 │
│  코드에서 비즈니스 규칙 추출 →     │
│  정책서 / 기획서 / 용어집 /       │
│  정적 웹사이트                   │
└─────────────────────────────────┘
```

1단계는 **개발자를 위한 구조 문서**를 만들고, 2단계는 그 위에 **전원이 공유할 수 있는 비즈니스 문서**를 만든다. 두 단계는 **기능 ID**(`{app}.{기능명_스네이크}`)를 공유하여 연결된다.

---

## 기능 ID: 두 체계의 연결 고리

코드 지식 체계의 FEATURES.md와 비즈니스 지식 체계의 POLICIES.md는 동일한 기능 ID를 섹션 헤딩으로 사용한다.

```
FEATURES.md                          POLICIES.md
┌────────────────────────┐          ┌────────────────────────┐
│ ## poker.투표           │ ◄──────► │ ## poker.투표           │
│ - 코드 위치: ...        │  기능 ID  │ - 규칙: 2명 이상 ...    │
│ - 트리거: ...           │  공유     │ - 위반 시: ...          │
│ - 영향 범위: ...        │          │ - 에러: ...             │
└────────────────────────┘          └────────────────────────┘
     (개발자용)                          (전원 공유)
```

- 코드 변경 → FEATURES.md 갱신 (기능 ID 단위) → 변경된 기능 ID 기준으로 POLICIES.md 해당 섹션만 재추출
- 기능 ID가 바뀌지 않은 섹션은 건드리지 않는다 (수동 보강 내용 보호)

---

## 산출물 전체 목록

### 1단계: 코드 지식 체계

| 산출물 | 역할 | 핵심 질문 |
|---|---|---|
| CODE_STRUCTURE.md | 코드 물리 구조 ↔ 도메인 매핑 | "이 도메인 코드는 어디에?" |
| DOMAIN_MAP.md | 전체 기능 간 영향 관계 조감도 | "이 기능 수정하면 어디 영향?" |
| FEATURES.md (도메인별) | 기능별 상세 영향 관계 + 체크리스트 | "구체적으로 뭘 확인해야 하지?" |
| ENTITIES.md (도메인별) | 엔티티 정의, 필드, 상태 전이 | "데이터 구조가 어떻게 되어 있지?" |
| OVERVIEW.md (도메인별) | 도메인 개요 | "이 도메인은 뭘 하는 곳?" |
| DEPENDENCY_MATRIX.md | 도메인 간 의존 매트릭스 | "도메인 간 커플링이 어느 정도?" |
| SERVICE_FLOW.md | 주요 유저 시나리오의 기능 흐름 | "이 흐름에서 내가 건드리는 부분은?" |
| CHANGE_LOG.md (도메인별) | 도메인 변경 이력 | "최근에 뭐가 바뀌었나?" |

### 2단계: 비즈니스 지식 체계

| 산출물 | 역할 | 핵심 질문 |
|---|---|---|
| POLICY_CATALOG.md | 전체 정책 색인 | "이 규칙 어디에 있지?" |
| POLICIES.md (도메인별) | 도메인별 비즈니스 규칙 카탈로그 | "이 기능의 정책이 뭐지?" |
| SPEC.md (도메인별) | 도메인별 기획서 | "이 기능은 왜 이렇게 동작하지?" |
| GLOSSARY.md (전체/도메인별) | 용어 정의 및 통일 | "이 용어가 정확히 무슨 뜻이지?" |
| _site/ | 정적 웹사이트 | 위 문서들을 검색·탐색 가능한 형태로 통합 |

### 핵심 원칙: 비즈니스 문서에 코드를 넣지 않는다

비즈니스 지식 체계의 최종 산출물(POLICIES.md, SPEC.md, GLOSSARY.md, 웹사이트)에는 **파일 경로, 라인 번호, 변수명, 코드 스니펫을 포함하지 않는다.** 코드 위치 추적은 기능 ID를 통해 코드 지식 체계(FEATURES.md)로 간접 참조한다.

---

## 대상 독자별 활용

| 독자 | 1단계 (코드 지식) | 2단계 (비즈니스 지식) |
|---|---|---|
| **개발자** | DOMAIN_MAP → FEATURES → DEPENDENCY_MATRIX 순으로 영향 범위 확인 후 코드 수정 | 기능 ID로 POLICIES.md 참조하여 비즈니스 맥락 파악 |
| **기획자** | (직접 사용하지 않음) | SPEC.md로 기능 이해, POLICIES.md로 규칙 확인 |
| **신규 팀원** | CODE_STRUCTURE → OVERVIEW로 전체 구조 파악 | GLOSSARY로 용어 학습, SPEC.md로 기능 파악 |
| **CS 담당자** | (직접 사용하지 않음) | POLICIES.md의 에러 시나리오 + 대응 방법 참조 |
| **Claude Code** | 코드 수정 전 영향 범위 사전 파악 | 비즈니스 규칙 준수 여부 사전 확인 |

---

## 사용 가능한 Skill

| 명령 | 설명 | 전제 조건 |
|---|---|---|
| `/oh-my-project-knowledge:init` | 코드베이스를 분석하여 코드 지식 문서 전체를 초기 생성 | 플러그인 설치 |
| `/oh-my-project-knowledge:update` | 코드 변경 후 코드 지식 문서를 증분 갱신 | 코드 지식 문서가 이미 존재 |
| `/oh-my-project-knowledge:biz-init` | 코드 지식 문서를 입력으로 비즈니스 지식 문서 전체를 초기 생성 | 코드 지식 문서 존재 |
| `/oh-my-project-knowledge:biz-update` | 코드 변경 후 비즈니스 지식 문서를 증분 갱신 | 비즈니스 지식 문서가 이미 존재 |
| `/oh-my-project-knowledge:context` | 특정 기능의 영향 범위와 비즈니스 규칙을 한번에 로딩 | 코드/비즈니스 지식 문서가 이미 존재 |

### 실행 순서

```
1. 플러그인 설치
2. /oh-my-project-knowledge:init      ← 코드 지식 체계 생성
3. /oh-my-project-knowledge:biz-init  ← 비즈니스 지식 체계 생성
4. (개발 중) /oh-my-project-knowledge:context {도메인}.{기능}
5. (코드 변경 후) /oh-my-project-knowledge:update → :biz-update
```

---

## 파일 배치 구조

### 플러그인 (oh-my-project-knowledge)

```
oh-my-project-knowledge/
├── .claude-plugin/
│   └── plugin.json                        # 매니페스트
├── commands/                              # 슬래시 명령
│   ├── init.md
│   ├── update.md
│   ├── biz-init.md
│   ├── biz-update.md
│   └── context.md
├── agents/                                # Subagent 정의
│   ├── domain-analyzer.md
│   └── policy-extractor.md
├── skills/                                # 형식 정의 (자동 참조)
│   ├── document-spec/SKILL.md
│   └── business-spec/SKILL.md
├── knowledge-system-guide.md              # ← 이 문서 (컨셉/목적)
└── knowledge-system-examples.md           # 산출물 예시
```

### 프로젝트 (명령 실행으로 자동 생성)

```
project-root/
├── CLAUDE.md                              # 기존 + 지식 문서 안내 블록 추가
└── .claude/
    └── docs/                              # ↓ 명령 실행으로 자동 생성
        ├── CODE_STRUCTURE.md
        ├── DOMAIN_MAP.md
        ├── SERVICE_FLOW.md
        ├── DEPENDENCY_MATRIX.md
        ├── domains/
        │   └── {domain-name}/
        │       ├── OVERVIEW.md
        │       ├── ENTITIES.md
        │       ├── FEATURES.md
        │       └── CHANGE_LOG.md
        └── business/
            ├── POLICY_CATALOG.md
            ├── GLOSSARY.md
            ├── domains/
            │   └── {domain-name}/
            │       ├── POLICIES.md
            │       ├── SPEC.md
            │       └── GLOSSARY.md
            └── _site/
                └── index.html

```

---

## 문서 신뢰도 유지 원칙

1. **코드가 진실의 원천** — 문서는 코드에서 추출한 파생물. 코드와 문서가 충돌하면 코드가 맞다.

2. **코드 지식 선행 갱신** — 순서: 코드 변경 → `/project-knowledge-update` → `/business-knowledge-update`. 기능 ID가 두 단계를 연결한다.

3. **수동 보강 보호** — `[수동 추가]` 태그가 붙은 내용은 자동 갱신 시 보존된다. 코드에서 추출 불가능한 비즈니스 배경을 직접 기록할 수 있다.

4. **점진적 구축** — 처음부터 완벽한 문서를 만들 필요 없다.
   - 1단계: DOMAIN_MAP + CODE_STRUCTURE (전체 윤곽)
   - 2단계: 주요 도메인의 FEATURES (영향 관계)
   - 3단계: DEPENDENCY_MATRIX + SERVICE_FLOW (교차 분석)
   - 4단계: POLICIES (비즈니스 규칙) → GLOSSARY (용어 통일) → SPEC (기획서) → 웹사이트

---

## CLAUDE.md에 삽입할 블록

아래 블록을 프로젝트의 CLAUDE.md에 붙여넣어 Claude Code가 지식 문서를 인식하도록 한다.

```markdown
## 프로젝트 지식 문서

이 프로젝트의 도메인 구조, 서비스 흐름, 의존성, 비즈니스 규칙은 아래 문서에 정리되어 있다.

### 코드 지식 (개발자용)
- 도메인 전체 맵: .claude/docs/DOMAIN_MAP.md
- 서비스 흐름: .claude/docs/SERVICE_FLOW.md
- 코드 구조: .claude/docs/CODE_STRUCTURE.md
- 의존성 매트릭스: .claude/docs/DEPENDENCY_MATRIX.md
- 도메인별 상세: .claude/docs/domains/{domain-name}/

### 비즈니스 지식 (전원 공유)
- 정책 카탈로그: .claude/docs/business/POLICY_CATALOG.md
- 전체 용어집: .claude/docs/business/GLOSSARY.md
- 도메인별 상세: .claude/docs/business/domains/{domain-name}/
- 웹사이트: .claude/docs/business/_site/index.html

## 코드 수정 전 영향 범위 확인 원칙

수정 대상 기능이 속한 도메인을 .claude/docs/DOMAIN_MAP.md에서 확인하라.
해당 기능의 `영향을 주는 곳 →` 열에 다른 기능이 있으면 반드시 영향 범위를 파악한 뒤 작업을 시작하라.

영향받는 기능이 존재하면 .claude/docs/domains/{도메인}/FEATURES.md를 열어 아래를 확인하라:
- impacts의 트리거 조건 — 내 수정이 이 트리거에 해당하는가
- impacts의 영향 범위 — 어떤 화면/데이터/로직이 영향받는가
- 변경 시 체크리스트 — 수정 후 반드시 검증할 항목

교차 도메인 영향이 의심되면 .claude/docs/DEPENDENCY_MATRIX.md에서 ●(강한 의존) 관계를 확인하라.

## 비즈니스 규칙 확인 원칙

새로운 validation, 에러 처리, 상수, 상태 전이를 추가할 때:
1. 해당 도메인의 POLICIES.md를 확인하여 기존 정책과 충돌하지 않는지 검토하라
2. 추가한 규칙을 POLICIES.md에 반영하라 (POL-{NNN} 형식)
3. 새 에러 메시지를 추가했다면 ERR-{NNN}으로 등록하라
```
