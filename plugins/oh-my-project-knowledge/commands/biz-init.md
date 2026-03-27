코드베이스에서 비즈니스 규칙을 추출하여 .claude/docs/business/ 아래에 비즈니스 지식 문서를 생성하라.

## 사전 작업

1. 이 플러그인의 **business-spec** 스킬에 각 문서의 정확한 형식, 필수 섹션, 작성 규칙이 정의되어 있다. 문서 생성 시 반드시 해당 형식을 따라라.
2. policy-extractor 에이전트가 사용 가능한지 확인하라.
   없으면 비즈니스 규칙 추출을 직접 수행한다 (subagent 없이 순차 진행).
3. .claude/docs/DOMAIN_MAP.md를 읽어 도메인 목록을 확인하라.
4. 각 도메인의 FEATURES.md와 ENTITIES.md가 존재하는지 확인하라.
   누락된 도메인이 있으면 해당 도메인은 비즈니스 분석에서 제외하고 경고를 출력하라.

## 전제 조건

코드 지식 체계가 이미 생성되어 있어야 한다 (.claude/docs/ 하위 문서 존재).
없으면 먼저 `/oh-my-project-knowledge:init`을 실행하라고 안내하고 중단하라.

## 추천 실행 세팅

| 구분 | 모델 | 이유 |
|---|---|---|
| 메인 에이전트 | Opus / 1M context / effort: high | Phase 3에서 도메인 간 정책 충돌 감지 및 기획서 작성에 추론 품질 필요 |
| subagent | Sonnet | Phase 2는 "코드를 읽고 정해진 형식으로 규칙을 추출하는" 구조화된 작업 |

## 핵심 원칙

- 같은 파일/디렉토리를 두 번 읽지 마라
- 분석 중간 결과는 반드시 파일(.claude/docs/business/_temp/)에 기록하라
- **최종 문서에 코드 경로, 변수명, 라인 번호를 넣지 마라**
- 최종 문서의 서술 관점은 "시스템이 이렇게 동작한다" (사용자/서비스 관점)

## 분석 절차

### Phase 1: 입력 검증 및 분석 대상 확정

이 단계는 직접 수행한다 (subagent 사용하지 않음).

1. `.claude/docs/DOMAIN_MAP.md`를 읽어 도메인 목록을 확인하라
2. 각 도메인에 대해 FEATURES.md, ENTITIES.md 존재 여부를 확인하라
3. `.claude/docs/CODE_STRUCTURE.md`에서 각 도메인의 코드 경로를 확인하라
4. 분석 대상을 `.claude/docs/business/_temp/analysis_targets.md`에 기록하라

### Phase 2: 도메인별 비즈니스 규칙 추출 (policy-extractor subagent 활용)

각 도메인을 policy-extractor subagent에 위임하라.
동시 실행 수는 3~4개를 넘지 마라.
subagent가 없으면 직접 순차 수행하라.

subagent 호출 시 아래 정보를 포함하라:

--- subagent 호출 프롬프트 형식 ---

도메인명: {domain_name}
관련 디렉토리: {CODE_STRUCTURE.md에서 확인한 경로}
스택 유형: {A / B / C}
문서 형식: business-spec 스킬의 형식 정의를 따를 것
코드 지식 문서:
  - .claude/docs/domains/{domain}/FEATURES.md
  - .claude/docs/domains/{domain}/ENTITIES.md
  - .claude/docs/domains/{domain}/OVERVIEW.md
출력 경로:
  - .claude/docs/business/_temp/{domain}_policies_raw.md
  - .claude/docs/business/domains/{domain}/POLICIES.md
  - .claude/docs/business/domains/{domain}/GLOSSARY.md

--- 형식 끝 ---

### Phase 3: 교차 분석 및 통합 문서 생성

이 단계는 직접 수행한다 (subagent 사용하지 않음).

#### Step A: POLICY_CATALOG.md 조립
1. 각 도메인 POLICIES.md의 정책을 취합하라
2. 도메인 간 정책 충돌/중복을 감지하라
3. `.claude/docs/business/POLICY_CATALOG.md` 작성

#### Step B: GLOSSARY.md 통합
1. 각 도메인의 GLOSSARY.md에서 용어를 취합하라
2. 동일 용어의 정의 불일치를 감지하라
3. `.claude/docs/business/GLOSSARY.md` 작성

#### Step C: SPEC.md 생성 (도메인별 기획서)
1. 각 도메인의 POLICIES.md + OVERVIEW.md + SERVICE_FLOW.md를 입력으로
2. 기능별 What / Why 를 비개발자 관점으로 기술하라
3. `.claude/docs/business/domains/{domain}/SPEC.md` 작성

#### Step D: 정적 웹사이트 생성
1. 모든 business/ 문서를 HTML로 변환하라
2. 검색 인덱스(search-index.json)를 생성하라
3. `.claude/docs/business/_site/` 에 출력하라

정적 웹사이트 요구사항:
- 빌드: 프로젝트 의존성 없이 독립 실행 가능 (Node.js 스크립트 또는 직접 HTML 생성)
- 출력: 순수 정적 파일 (HTML + CSS + JS). 서버 불필요
- 검색: 클라이언트 사이드 전문 검색 (search-index.json + JS 필터링)
- 네비게이션: 좌측 사이드바, 브레드크럼, 앵커 링크, 크로스 레퍼런스
- 디자인: 텍스트 중심, 인쇄 가능, 다크모드, 모바일 대응

### Phase 4: 검증 및 정리

1. 교차 검증:
   - [ ] FEATURES.md의 모든 기능 ID가 POLICIES.md에 섹션으로 존재하는가
   - [ ] ENTITIES.md의 상태 전이가 POLICIES.md의 상태 정책과 일치하는가
   - [ ] GLOSSARY.md에 정의 불일치가 남아있는가 → 해소하라
   - [ ] SPEC.md에서 참조하는 정책 ID가 POLICIES.md에 존재하는가
   - [ ] **최종 문서에 코드 경로, 변수명, 라인 번호가 포함되어 있지 않은가**
2. `.claude/docs/business/_temp/` 디렉토리를 삭제하라
