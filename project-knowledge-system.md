# Claude Code 프로젝트 지식 체계 (Project Knowledge System)

---

## 1. 개요

### 해결하고자 하는 문제

프로젝트 규모가 커지면 "이 기능을 수정하면 어디에 영향이 가는가?"를 파악하기 어렵다. 코드베이스를 처음 보는 사람은 물론, 오래 참여한 개발자도 도메인 간 의존 관계를 놓치고 사이드 이펙트를 만든다.

이 문서는 Claude Code가 코드베이스를 분석하여 **도메인 구조, 기능 간 영향 관계, 서비스 흐름**을 문서화하고, 이후 개발 시 그 문서를 참조하여 영향 범위를 사전에 파악하는 체계를 정의한다.

### 핵심 목표

- **새 기능 개발 시** → 수정 대상의 영향 범위를 코드 수정 전에 파악
- **리팩토링 시** → 도메인 간 의존성 매트릭스로 연쇄 영향 추적

### 프로젝트에 배치하는 파일

```
project-root/
├── CLAUDE.md                              # 프로젝트 루트 (기존 + 코드 수정 전 원칙 추가)
└── .claude/
    ├── MEMORY.md                          # 스택, 컨벤션, 명령어
    ├── DOCUMENT_SPEC.md                   # 문서 형식 정의서 (설정 파일)
    ├── agents/
    │   └── domain-analyzer.md             # subagent 정의 (3장 참조)
    └── docs/                              # ↓ 아래는 모두 프롬프트 실행으로 생성
        ├── DOMAIN_MAP.md
        ├── SERVICE_FLOW.md
        ├── CODE_STRUCTURE.md
        ├── DEPENDENCY_MATRIX.md
        └── domains/
            └── {domain-name}/
                ├── OVERVIEW.md
                ├── ENTITIES.md
                ├── FEATURES.md
                └── CHANGE_LOG.md
```

### 기존 작업 원칙 md에 추가할 내용

아래 블록을 기존 작업 원칙 md (또는 CLAUDE.md)에 삽입한다.
"문서 위치 안내"와 "코드 수정 전/후 규칙"을 하나의 블록으로 통합했다.
상세 내용은 5장 운영 가이드에서 제공한다.

---

## 2. 핵심 문서 상세 스펙

> 각 문서의 상세 형식 정의는 `.claude/DOCUMENT_SPEC.md`에 별도 관리한다.
> 여기서는 각 문서의 역할과 문서 간 관계를 설명한다.

### 문서 간 관계

| 문서 | 역할 | 참조 방향 |
|---|---|---|
| DOMAIN_MAP.md | 전체 조감도 — "이 기능 수정하면 어디 영향?" | → FEATURES.md로 상세 확인 |
| FEATURES.md | 기능별 상세 — "구체적으로 뭘 확인해야 하지?" | ← DOMAIN_MAP.md의 기능 ID와 1:1 |
| DEPENDENCY_MATRIX.md | 매트릭스 뷰 — "도메인 간 커플링 한눈에" | ← FEATURES.md의 외부 참조를 취합 |
| SERVICE_FLOW.md | 흐름 뷰 — "이 흐름에서 내가 건드리는 부분 어디?" | ← DOMAIN_MAP.md + DEPENDENCY_MATRIX.md |
| CODE_STRUCTURE.md | 물리 구조 — "이 도메인 코드는 어디에?" | → 모든 문서의 코드 경로 기준 |
| ENTITIES.md | 엔티티 정의 — "데이터 구조와 상태 전이" | ← FEATURES.md에서 참조 |
| OVERVIEW.md | 도메인 개요 — "이 도메인은 뭘 하는 곳?" | 진입점 역할 |
| CHANGE_LOG.md | 변경 이력 — "최근에 뭐가 바뀌었나?" | ← 갱신 프롬프트로 기록 |

### DOMAIN_MAP.md 예시

```markdown
# Domain Map

> 최종 갱신: 2025-06-15

## 도메인 맵

### 채용 (recruitment)

| 기능 ID | 영향을 주는 곳 → | ← 영향을 받는 곳 |
|---|---|---|
| backoffice.포지션_생성_수정 | → public.공고_상세, public.지원서_제출 | ← (없음) |
| public.공고_상세 | → public.지원서_제출 | ← backoffice.포지션_생성_수정 |

## 교차 도메인 영향 관계 (Cross-Domain Impact)

| 출발 | 도착 | 영향 유형 | 설명 |
|---|---|---|---|
| backoffice.조직도_관리 | backoffice.평가대상_설정 | 데이터 의존 | 조직 변경 시 평가 대상 자동 재설정 |
```

### FEATURES.md 예시

```markdown
## backoffice.포지션_생성_수정

- **설명**: 채용 포지션을 생성하거나 수정하는 관리자 기능
- **코드 위치**:
  - FE: `src/apps/backoffice/recruitment/position/`
  - BE: `src/domain/recruitment/position/`
  - API: `POST /api/backoffice/positions`, `PUT /api/backoffice/positions/{id}`
- **주요 엔티티**: Position, PositionRequirement, JobCategory
- **영향을 줌 (impacts)**:
  - `public.공고상세` — 포지션 데이터가 공고 페이지에 표시됨
    - 트리거: 포지션 상태가 'published'로 변경될 때
    - 영향 범위: 공고 제목, 요구사항, 마감일
- **영향을 받음 (affected_by)**:
  - (없음 — 최상위 생성 기능)
- **변경 시 체크리스트**:
  - [ ] Position 엔티티 스키마 변경 시 → public.공고상세 렌더링 확인
  - [ ] 상태 전이 로직 변경 시 → 공고 노출 조건 확인
```

### DEPENDENCY_MATRIX.md 예시

```markdown
> 행(→) = 영향을 주는 도메인, 열(←) = 영향을 받는 도메인
> ●: 강한 의존 (데이터/로직 직접 참조)
> ○: 약한 의존 (이벤트/알림 수준)

|  | recruitment | hr | evaluation | approval |
|---|---|---|---|---|
| **recruitment** | - | - | - | ○ |
| **hr** | - | - | ● | ○ |
| **evaluation** | - | ○ | - | ● |
```

### SERVICE_FLOW.md 예시

```markdown
## FLOW-001: 채용 공고 게시 → 지원 → 평가

### 흐름

1. **[backoffice] 포지션 생성** → Position (status: draft)
2. **[backoffice] 포지션 게시** → status → published → public.공고상세 (JobPosting 자동 생성)
3. **[public] 공고 조회 & 지원서 제출** → Application (status: submitted)
4. **[backoffice] 서류 검토 & 평가** → evaluation 도메인의 평가 엔진 호출

### 흐름 내 수정 영향 분석

| 수정 지점 | 영향받는 후속 단계 | 확인 사항 |
|---|---|---|
| Step 1 (포지션 스키마) | Step 2, 3 | 공고 표시 필드, 지원서 필수항목 |
```

---

## 3. Subagent 설정 및 추천 실행 세팅

### 추천 세팅

| 구분 | 모델 | 이유 |
|---|---|---|
| 메인 에이전트 | Opus 4.6 / 1M context / effort: high | Phase 3 교차 분석에서 도메인 간 연쇄 의존 추론 필요 |
| subagent | Sonnet | Phase 2는 "파일을 읽고 정해진 형식으로 추출하는" 구조화된 작업. 비용 대비 효율적 |

subagent를 Opus로 올리고 싶다면 domain-analyzer.md의 model을 opus로 변경한다.
도메인이 5개 이하이고 Kafka/이벤트 기반 의존이 많은 BE 프로젝트에서 효과적이다.

### domain-analyzer subagent 정의

아래 파일을 `.claude/agents/domain-analyzer.md`에 저장한다.

```markdown
---
name: domain-analyzer
description: |
  도메인 단위 코드 분석 전용 에이전트.
  메인 에이전트가 도메인 경로와 스택 유형을 전달하면,
  해당 도메인의 기능/엔티티/의존성을 분석하여
  _temp/{domain}_raw.md 파일로 출력한다.
  "도메인 분석", "domain analyze" 키워드가 포함된 작업에 사용하라.
tools:
  - Read
  - Grep
  - Glob
  - Write
  - Bash(grep:*,find:*,wc:*,head:*,tail:*,cat:*)
model: sonnet
---

당신은 도메인 분석 전문 에이전트이다.
메인 에이전트로부터 아래 정보를 전달받는다:

- 분석 대상 도메인명
- 도메인 관련 디렉토리 경로 목록
- 스택 유형 (A: BE 중심 / B: FE 중심 / C: 모노레포)
- DOCUMENT_SPEC.md 경로
- 공유 영역 제외 목록 (분석 범위에서 제외할 경로)

## 임무

전달받은 도메인의 코드를 탐색하여 아래 4가지 산출물을 생성하라.

### 산출물 1: _temp/{domain_name}_raw.md (필수)

아래 형식을 정확히 따라라. 이 파일이 메인 에이전트의 Phase 3 입력이 된다.
--- 형식 시작 ---

## {domain_name} 분석 결과

### 기능 목록
- {app}.{기능_ID}: {설명}
  - FE: {경로} (해당 시)
  - BE: {경로} (해당 시)
  - API: {METHOD} {endpoint} (해당 시)

### 엔티티
- {EntityName}: {파일 경로}
  - 주요 필드: {필드 목록}
  - 상태 전이: {있으면 기술, 없으면 "없음"}

### 외부 참조 (이 도메인이 import하는 다른 도메인)
- {other_domain}.{Entity/Service}: {import 경로}
  - 참조 유형: {직접 import / 이벤트 구독 / API 호출 / DB 외래키 / 캐시 참조}
  - 의존 강도: {● 강함 / ○ 약함}

### 이벤트/메시지 (해당 시)
- 발행: {토픽/이벤트명} → {어떤 데이터를 발행하는가}
- 구독: {토픽/이벤트명} → {어떤 처리를 하는가}

--- 형식 끝 ---

### 산출물 2: domains/{domain_name}/OVERVIEW.md
DOCUMENT_SPEC.md의 OVERVIEW 형식을 따라 작성하라.

### 산출물 3: domains/{domain_name}/ENTITIES.md
DOCUMENT_SPEC.md의 ENTITIES 형식을 따라 작성하라.

### 산출물 4: domains/{domain_name}/FEATURES.md
DOCUMENT_SPEC.md의 FEATURES 형식을 따라 작성하라.
단, 다른 도메인으로부터의 영향(affected_by) 중
이 도메인 내에서 확인 불가능한 것은 `[교차 분석 시 보완]`으로 표기하라.

## 스택별 탐색 규칙

### 스택 유형 A (BE 중심: Spring Boot/Kotlin, Express, NestJS, Go 등)

프레임워크에 따라 디렉토리 구조가 다르므로, 아래에서 해당하는 패턴을 따르라.

API 엔드포인트 추출:
  - Spring Boot: controller 클래스의 @RequestMapping, @GetMapping, @PostMapping 등
  - Express/NestJS: router 파일 또는 @Controller 데코레이터
  - 위에 해당하지 않으면 라우트 등록 파일(routes.kt, router.ts 등)에서 추출하라
  - 엔드포인트를 도메인별로 그루핑할 때, URL 경로의 1~2depth가 기준이 된다
    (예: /api/recruitment/positions → recruitment 도메인)

로직 식별:
  - service/{도메인}/ — 핵심 비즈니스 로직
  - usecase/{도메인}/ — 유스케이스 계층 (있는 경우)
  - repository/{도메인}/ — 데이터 접근 계층
  - config/{도메인}/ — 도메인별 설정 (Bean 정의, 프로퍼티 등)
  - scheduler/ 또는 batch/ — 정기 실행 작업 (어떤 도메인의 데이터를 처리하는지 확인)
  - listener/ 또는 handler/ — 이벤트/메시지 처리 (어떤 도메인의 이벤트를 소비하는지 확인)
  - client/ 또는 adapter/ — 외부 시스템 연동 계층

데이터 구조:
  - entity/ 또는 model/ — JPA @Entity, data class, POJO 등
  - dto/ — 요청/응답 데이터 전송 객체
  - enum/ — 상태값, 타입 코드 정의
  - 상태 전이 파악: entity 내 status/state 필드를 찾고,
    service 계층에서 해당 필드를 변경하는 메서드를 추적하라
    (예: `position.publish()`, `application.updateStatus(...)`)

의존성 추적:
  - import/require 관계 (service → 다른 도메인 service/repository 참조)
  - 이벤트 기반 의존 (Kafka/RabbitMQ):
    grep -r "KafkaTemplate\|@KafkaListener\|@SendTo\|ProducerRecord" 으로 발행/구독 패턴 탐색
    토픽명에서 도메인 관계를 추출 (예: "recruitment-position-created" 토픽)
  - Spring Event:
    grep -r "ApplicationEventPublisher\|@EventListener\|@TransactionalEventListener"
    같은 프로세스 내 도메인 간 이벤트 발행/구독 패턴 탐색
  - Redis 캐시 의존:
    @Cacheable, @CacheEvict, RedisTemplate 사용처에서
    다른 도메인의 데이터를 캐싱하거나 무효화하는 패턴 탐색
  - DB 외래키: 엔티티 간 @ManyToOne, @OneToMany, @JoinColumn 관계에서 교차 도메인 참조 확인
  - FeignClient / RestTemplate / WebClient: 다른 내부 서비스를 호출하는 패턴 (MSA 구조 시)

### 스택 유형 B (FE 중심: Next.js, Nuxt.js, Vue, React SPA 등)

프레임워크에 따라 디렉토리 구조가 다르므로, 아래에서 해당하는 패턴을 따르라.

화면 단위 추출:
  - Next.js: pages/{도메인}/ 또는 app/{도메인}/
  - Nuxt.js: pages/{도메인}/
  - Vue (SFC): views/{도메인}/ 또는 pages/{도메인}/
  - React SPA: pages/{도메인}/, routes/{도메인}/, 또는 라우터 설정 파일에서 추출
  - 위에 해당하지 않으면 라우터 설정 파일(router.js, routes.ts 등)에서 화면 목록을 추출하라

로직 식별:
  - components/{도메인}/ — 도메인별 컴포넌트
  - hooks/{도메인}/ 또는 composables/{도메인}/ — 재사용 로직 (React: hooks, Vue3: composables)
  - store/{도메인}/ 또는 stores/{도메인}/ — 상태 관리 (Vuex, Pinia, Redux, Zustand)
  - services/{도메인}/ 또는 api/{도메인}/ — API 호출 계층
  - mixins/{도메인}/ — Vue2 mixins (해당 시)

데이터 구조:
  - types/, models/, interfaces/ 디렉토리 또는 각 도메인 폴더 내 types.ts, *.d.ts
  - Vue2의 경우 컴포넌트 data() / props 정의에서 추출

의존성 추적:
  - import 문 (다른 도메인 컴포넌트/훅/유틸 참조)
  - props 전달 체인: 부모 → 자식 컴포넌트로 전달되는 데이터
  - 이벤트 전달: $emit (Vue), CustomEvent, EventBus 등 자식 → 부모 또는 형제 간 통신
  - URL 파라미터/쿼리 의존: 라우터를 통한 페이지 간 데이터 전달 (router.push, Link, router-link 등)
  - 공유 상태: Vuex/Pinia store, Context, Zustand, Redux 등 전역 상태를 통한 도메인 간 연결
  - BE API 호출: fetch/axios 호출 URL 패턴에서 도메인 간 데이터 의존 추출

### 스택 유형 C (모노레포: Turborepo, Nx, Lerna, pnpm workspace 등)

모노레포는 여러 앱과 공유 패키지가 한 저장소에 있다.
분석이 2계층(앱 → 도메인)이므로, 아래 순서를 따르라.

앱/패키지 구조 파악 (Phase 1에서 수행):
  - apps/ 하위: 각 디렉토리가 독립 앱 (예: apps/backoffice, apps/public, apps/admin)
  - packages/ 하위: 공유 패키지. 아래 두 종류를 구분하라:
    - 범용 공유 패키지: ui, utils, config, eslint-config 등 → 공유 영역 제외 목록에 추가
    - 도메인 전용 패키지: {도메인}-types, {도메인}-api, {도메인}-shared 등 → 해당 도메인의 관련 디렉토리에 포함
  - 판단 기준: 패키지명에 도메인명이 포함되어 있거나, 특정 도메인의 앱에서만 참조하면 도메인 전용

도메인 경계 식별:
  - 각 앱(apps/*) 내부에서 유형 B의 기준을 적용하여 도메인을 식별하라
  - 기능 ID는 `{앱명}.{기능명_스네이크}` 형식
    (예: backoffice.포지션_생성_수정, public.공고_상세)
  - 같은 도메인이 여러 앱에 걸쳐 있을 수 있다 (예: recruitment 도메인이 backoffice와 public 양쪽에 존재)

화면/로직/데이터 구조:
  - 각 앱 내부는 유형 B의 탐색 규칙을 그대로 적용하라
  - 도메인 전용 패키지의 내용은 해당 도메인의 엔티티/타입으로 포함하라

의존성 추적:
  - 앱 내부 의존성: 유형 B의 의존성 추적 규칙을 적용
  - 패키지 간 의존성: 각 앱/패키지의 package.json dependencies를 확인하라
    - 어떤 앱이 어떤 패키지를 참조하는지 매핑
    - 도메인 전용 패키지를 여러 앱이 참조하면 → 교차 앱 의존으로 기록
  - workspace 프로토콜: "workspace:*" 의존은 로컬 패키지 참조이므로 import와 동일하게 취급
  - turbo.json / nx.json의 pipeline/task 의존: 빌드 순서에서 암묵적 의존 관계를 유추할 수 있다 (참고 수준)

## 규칙
- 코드에서 확인되지 않는 관계는 `[미확인]`으로 표기. 추측 금지.
- 기능 ID는 `{app}.{기능명_스네이크}` 형식.
- 파일 탐색은 최소한으로. 같은 파일을 두 번 읽지 마라.
- 공유 영역 제외 목록에 포함된 경로는 탐색하지 마라.
  단, 자기 도메인 코드가 공유 영역을 import하는 것은 "외부 참조"에 기록하라:
  예: `[shared] auth/permission.ts: {import 경로}` — 참조 유형: 공유 모듈
- 공유 영역에 위치한 코드의 기능/엔티티를 자기 도메인의 산출물로 작성하지 마라.
- 완료 시 메인 에이전트에게 "✅ {domain_name} 분석 완료. 기능 {N}개, 엔티티 {M}개, 외부 참조 {K}건" 형태로 보고하라.
```

---

## 4. 주요 프롬프트

### 4-1. 초기 생성 프롬프트

> 전제 조건: `.claude/DOCUMENT_SPEC.md`와 `.claude/agents/domain-analyzer.md`가 존재해야 한다.

```
프로젝트 코드베이스를 분석하여 .claude/docs/ 아래에 프로젝트 지식 문서를 생성하라.

## 사전 작업
1. .claude/DOCUMENT_SPEC.md 파일을 읽어라.
   이 파일에 각 문서의 정확한 형식, 필수 섹션, 작성 규칙이 정의되어 있다.
2. .claude/agents/domain-analyzer.md 가 존재하는지 확인하라.
   없으면 도메인 분석을 직접 수행한다 (subagent 없이 순차 진행).

## 핵심 원칙: 컨텍스트 절약

- 같은 파일/디렉토리를 두 번 읽지 마라
- 분석 중간 결과는 반드시 파일(.claude/docs/_temp/)에 기록하라
- 한 단계가 끝나면 중간 결과 파일을 참조하라. 이전 단계의 탐색 내용을 기억에 의존하지 마라

## 분석 절차

### Phase 1: 전체 윤곽 파악 → CODE_STRUCTURE.md 생성

이 단계는 직접 수행한다 (subagent 사용하지 않음).

1. `tree -L 3 -d --charset=ascii` 로 디렉토리 구조를 파악하라
2. 앱 단위를 식별하라 (backoffice, public, admin, api 등)
3. 도메인 경계를 식별하라 (디렉토리명, 패키지 구조, 모듈 분리 기준)
4. 스택 유형을 판단하라:
   - 유형 A (BE 중심): controller, service, entity 디렉토리가 존재
   - 유형 B (FE 중심): pages/, views/, components/, router 설정 등 FE 라우팅/뷰 구조가 존재
   - 유형 C (모노레포): apps/ + packages/ 구조, turbo.json/nx.json/pnpm-workspace.yaml 존재
5. 공유 영역을 식별하고 분리하라:
   - shared/, common/, lib/, utils/, core/ 등 여러 도메인이 공유하는 경로를 찾아라
   - 이 경로들은 개별 도메인의 분석 범위에서 제외한다
   - CODE_STRUCTURE.md의 "공유 모듈" 섹션에 기록하라
6. 결과를 즉시 `.claude/docs/CODE_STRUCTURE.md`로 작성하라 (DOCUMENT_SPEC.md 형식)
7. 도메인 목록을 `.claude/docs/_temp/domain_list.md`에 기록하라:

--- domain_list.md 형식 ---

## 분석 대상 도메인
- {domain_name}: {관련 디렉토리 경로 목록}
...

## 스택 유형: {A 또는 B 또는 C}

## 공유 영역 (subagent 분석 범위 제외)
- {shared_path_1}: {설명}
- {shared_path_2}: {설명}

--- 형식 끝 ---

이 단계에서는 파일 내부를 읽지 마라. 디렉토리 구조만으로 판단하라.

### Phase 2: 도메인별 분석 (domain-analyzer subagent 활용)

`.claude/docs/_temp/domain_list.md`를 읽고, 각 도메인을 domain-analyzer subagent에 위임하라.

각 도메인에 대해 domain-analyzer subagent를 호출하라.
호출 시 아래 정보를 프롬프트에 포함하라:

--- subagent 호출 프롬프트 형식 ---

도메인명: {domain_name}
관련 디렉토리:
  - {경로1}
  - {경로2}
스택 유형: {A 또는 B 또는 C}
DOCUMENT_SPEC 경로: .claude/DOCUMENT_SPEC.md
공유 영역 제외 목록:
  - {shared_path_1}
  - {shared_path_2}
출력 경로:
  - .claude/docs/_temp/{domain_name}_raw.md
  - .claude/docs/domains/{domain_name}/OVERVIEW.md
  - .claude/docs/domains/{domain_name}/ENTITIES.md
  - .claude/docs/domains/{domain_name}/FEATURES.md

--- 형식 끝 ---

도메인 간 분석은 독립적이므로, 가능한 한 여러 subagent를 동시에 실행하라.
단, 동시 실행 수는 3~4개를 넘지 마라.

subagent가 없으면 각 도메인에 대해 직접 순차 수행하라:
도메인 디렉토리 탐색(1회) → _raw.md 기록 → 문서 작성 → 다음 도메인.

모든 도메인의 _raw.md 파일이 생성되었는지 확인하라.
누락된 도메인이 있으면 해당 subagent를 재실행하라.

### Phase 3: 교차 분석 → 전체 문서 생성

이 단계는 직접 수행한다 (subagent 사용하지 않음).
`.claude/docs/_temp/` 의 중간 결과 파일들만 읽고 진행하라.

#### Step A: 의존성 매트릭스 조립
1. 각 `{domain}_raw.md`의 "외부 참조" 섹션을 취합하라
2. 의존 강도를 판단하라:
   - ● 강한 의존: 직접 import, DB 직접 참조, 동기 호출
   - ○ 약한 의존: 이벤트, 메시지 큐, 비동기 처리
3. `.claude/docs/DEPENDENCY_MATRIX.md` 작성

#### Step B: DOMAIN_MAP 조립
1. 각 `{domain}_raw.md`의 기능 목록 + 의존성 매트릭스를 결합하라
2. 기능별 `영향을 주는 곳 →` / `← 영향을 받는 곳` 매핑
3. `.claude/docs/DOMAIN_MAP.md` 작성

#### Step C: FEATURES.md 보완
1. `[교차 분석 시 보완]`으로 표기했던 impacts/affected_by를 확정하라
2. 트리거 조건, 영향 범위, 변경 시 체크리스트를 보강하라
3. 각 도메인의 `FEATURES.md`를 갱신

#### Step D: SERVICE_FLOW 도출
1. DOMAIN_MAP.md와 DEPENDENCY_MATRIX.md를 읽고 주요 흐름 최소 3개를 식별하라
   (판단 기준: 여러 도메인을 관통하는 흐름, ● 의존이 연쇄되는 흐름)
2. `.claude/docs/SERVICE_FLOW.md` 작성

### Phase 4: 검증 및 정리

1. 교차 검증:
   - [ ] DOMAIN_MAP.md의 "기능 ID" 열에 있는 모든 ID가 FEATURES.md에 동일한 헤딩(`## {기능_ID}`)으로 존재하는가
   - [ ] FEATURES.md의 모든 기능 ID가 DOMAIN_MAP.md에도 존재하는가 (양방향 대조)
   - [ ] DOMAIN_MAP.md의 교차 도메인 영향이 DEPENDENCY_MATRIX.md에 반영되어 있는가
   - [ ] SERVICE_FLOW.md에서 참조하는 기능 ID가 FEATURES.md에 존재하는가
   - [ ] 모든 impacts 관계에 대응하는 affected_by가 상대 도메인의 FEATURES.md에 존재하는가
   - [ ] 불일치가 있으면 수정하라
2. `.claude/docs/_temp/` 디렉토리를 삭제하라
```

### 4-2. 갱신 프롬프트 (기능 추가/수정 후)

```
{도메인명} 도메인에 변경이 발생했다.
코드 변경 사항을 분석하여 아래 문서를 갱신하라:

1. .claude/docs/domains/{domain-name}/FEATURES.md — 변경된 기능의 영향 관계 업데이트
2. .claude/docs/DOMAIN_MAP.md — 해당 도메인 행의 영향 관계 테이블 갱신
3. .claude/docs/DEPENDENCY_MATRIX.md — 새로운 교차 도메인 의존이 생겼다면 추가
4. .claude/docs/domains/{domain-name}/CHANGE_LOG.md — 변경 이력 추가

변경 사항: {간략한 변경 설명}
```

### 4-3. 개발 전 컨텍스트 로딩 프롬프트

```
지금부터 {도메인}.{기능} 을 수정/개발한다.
작업 시작 전에 아래 문서를 읽고 영향 범위를 정리하라:

1. .claude/docs/DOMAIN_MAP.md 에서 해당 기능의 영향 관계 확인
2. .claude/docs/domains/{domain}/FEATURES.md 에서 상세 영향 관계 + 체크리스트 확인
3. .claude/docs/DEPENDENCY_MATRIX.md 에서 교차 도메인 영향 확인
4. .claude/docs/SERVICE_FLOW.md 에서 해당 기능이 포함된 흐름 확인

위 분석 결과를 기반으로:
- 이 작업으로 영향받는 기능 목록
- 수정 후 확인해야 할 체크리스트
를 먼저 정리하고, 승인 후 개발을 시작하라.
```

---

## 5. 운영 가이드

### 문서 신뢰도 유지 원칙

1. **코드가 진실의 원천(Single Source of Truth)**
   - 문서는 코드를 설명하는 부산물. 코드와 문서가 충돌하면 코드가 맞다
   - 정기적으로 "갱신 프롬프트(4-2)"를 실행하여 동기화

2. **점진적 구축**
   - 처음부터 완벽한 문서를 만들려 하지 않는다
   - 1단계: DOMAIN_MAP.md + CODE_STRUCTURE.md (전체 윤곽)
   - 2단계: 주요 도메인의 FEATURES.md (영향 관계)
   - 3단계: DEPENDENCY_MATRIX.md + SERVICE_FLOW.md (교차 분석)

3. **변경 시 갱신 습관**
   - PR 머지 후 "갱신 프롬프트(4-2)" 실행을 팀 프로세스에 포함
   - 또는 CLAUDE.md에 "코드 변경 후 관련 문서 갱신" 규칙 추가

### 기존 작업 원칙 md에 삽입할 블록

아래 코드블록 전체를 기존 작업 원칙 md (또는 CLAUDE.md)에 붙여넣는다.

```markdown
## 프로젝트 지식 문서

이 프로젝트의 도메인 구조, 서비스 흐름, 의존성 정보는 아래 문서에 정리되어 있다.

- 도메인 전체 맵: .claude/docs/DOMAIN_MAP.md
- 서비스 흐름: .claude/docs/SERVICE_FLOW.md
- 코드 구조: .claude/docs/CODE_STRUCTURE.md
- 의존성 매트릭스: .claude/docs/DEPENDENCY_MATRIX.md
- 도메인별 상세: .claude/docs/domains/{domain-name}/

## 코드 수정 전 영향 범위 확인 원칙

### 필수 확인 절차

수정 대상 기능이 속한 도메인을 `.claude/docs/DOMAIN_MAP.md`에서 확인하라.
해당 기능의 `영향을 주는 곳 →` 열에 다른 기능이 있으면 반드시 영향 범위를 파악한 뒤 작업을 시작하라.

영향받는 기능이 존재하면 `.claude/docs/domains/{도메인}/FEATURES.md`를 열어 아래를 확인하라:
- `impacts`의 트리거 조건 — 내 수정이 이 트리거에 해당하는가
- `impacts`의 영향 범위 — 어떤 화면/데이터/로직이 영향받는가
- `변경 시 체크리스트` — 수정 후 반드시 검증할 항목

교차 도메인 영향이 의심되면 `.claude/docs/DEPENDENCY_MATRIX.md`에서 해당 도메인 행을 확인하라.
`●`(강한 의존) 관계가 있는 도메인은 반드시 함께 검토하라.

### 금지 사항

- 영향 범위를 확인하지 않고 코드를 수정하는 것을 금지한다.
- `DOMAIN_MAP.md`에 등록되지 않은 기능 간 의존 관계를 암묵적으로 생성하는 것을 금지한다. 새로운 의존이 필요하면 문서에 먼저 반영하라.
- 영향받는 기능의 동작을 확인하지 않고 PR을 올리는 것을 금지한다.

### 수정 후 문서 갱신 의무

코드 수정으로 영향 관계가 변경되었으면 아래 문서를 갱신하라:
- `DOMAIN_MAP.md` — 해당 기능 행의 영향 관계 테이블
- `FEATURES.md` — impacts/affected_by 및 체크리스트
- `DEPENDENCY_MATRIX.md` — 새로운 교차 도메인 의존이 생긴 경우
- `CHANGE_LOG.md` — 변경 이력 추가
```
