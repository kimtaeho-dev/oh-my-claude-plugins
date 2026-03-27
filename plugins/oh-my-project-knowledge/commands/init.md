프로젝트 코드베이스를 분석하여 .claude/docs/ 아래에 프로젝트 지식 문서를 생성하라.

## 사전 작업

1. 이 플러그인의 **document-spec** 스킬에 각 문서의 정확한 형식, 필수 섹션, 작성 규칙이 정의되어 있다. 문서 생성 시 반드시 해당 형식을 따라라.
2. domain-analyzer 에이전트가 사용 가능한지 확인하라.
   없으면 도메인 분석을 직접 수행한다 (subagent 없이 순차 진행).

## 추천 실행 세팅

| 구분 | 모델 | 이유 |
|---|---|---|
| 메인 에이전트 | Opus / 1M context / effort: high | Phase 3 교차 분석에서 도메인 간 연쇄 의존 추론 필요 |
| subagent | Sonnet | Phase 2는 "파일을 읽고 정해진 형식으로 추출하는" 구조화된 작업 |

subagent를 Opus로 올리고 싶다면 domain-analyzer.md의 model을 opus로 변경한다.
도메인이 5개 이하이고 Kafka/이벤트 기반 의존이 많은 BE 프로젝트에서 효과적이다.

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
6. 결과를 즉시 `.claude/docs/CODE_STRUCTURE.md`로 작성하라 (document-spec 스킬의 형식을 따라)
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
문서 형식: document-spec 스킬의 형식 정의를 따를 것
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
