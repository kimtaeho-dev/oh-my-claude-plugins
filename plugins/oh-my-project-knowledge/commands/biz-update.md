$ARGUMENTS 도메인에 변경이 발생했다.
코드 변경 사항을 분석하여 비즈니스 지식 문서를 갱신하라.

## 인자 형식

`{도메인명}` — 변경이 발생한 도메인 (예: poker, room, jira)
인자가 없으면 git diff로 변경된 파일을 분석하여 영향받은 도메인을 자동 판별하라.

## 전제 조건 확인

1. 코드 지식 체계의 갱신이 이미 완료되었는지 확인하라.
   .claude/docs/domains/{domain-name}/FEATURES.md가 최신 상태여야 한다.
   최신이 아니면 먼저 `/oh-my-project-knowledge:update`를 실행하라고 안내하라.
2. .claude/docs/business/domains/{domain-name}/POLICIES.md가 존재하는지 확인하라.
   없으면 먼저 `/oh-my-project-knowledge:biz-init`을 실행하라고 안내하라.

## 갱신 절차

### 1. 갱신 범위 판단

1. FEATURES.md에서 변경된 기능 ID를 식별하라
2. 해당 기능 ID에 대응하는 POLICIES.md 섹션만 재추출 대상으로 삼아라
3. 해당 기능 ID의 코드 위치(FEATURES.md에 기재된 경로)를 탐색하여 정책을 재추출하라

### 2. 수동 보강 보호 규칙

- POLICIES.md 또는 SPEC.md에 `[수동 추가]` 태그가 붙은 내용은 덮어쓰지 마라
- 재추출 대상 섹션에 `[수동 추가]` 내용이 있으면 보존한 채로 자동 추출 부분만 갱신하라
- 자동 추출 내용과 수동 추가 내용이 충돌하면 `[충돌 — 확인 필요]` 마킹 후 양쪽을 모두 남겨라

### 3. 갱신 대상 문서

1. `.claude/docs/business/domains/{domain-name}/POLICIES.md` — 변경된 기능 ID 섹션 재추출
2. `.claude/docs/business/POLICY_CATALOG.md` — 해당 도메인 섹션 갱신
3. `.claude/docs/business/domains/{domain-name}/GLOSSARY.md` — 새 용어 반영
4. `.claude/docs/business/GLOSSARY.md` — 전체 용어집 갱신
5. `.claude/docs/business/domains/{domain-name}/SPEC.md` — 기획서 갱신 (해당 시)
6. `.claude/docs/business/_site/` — 웹사이트 재빌드

### 4. 교차 검증

- [ ] 변경된 기능 ID의 POLICIES.md 섹션이 현재 코드와 일치하는가
- [ ] POLICY_CATALOG.md의 해당 도메인 행이 POLICIES.md와 일치하는가
- [ ] 새 용어가 GLOSSARY.md에 반영되었는가
- [ ] **갱신된 문서에 코드 경로, 변수명, 라인 번호가 포함되어 있지 않은가**

## 규칙

- 변경된 기능 ID에 해당하는 부분만 갱신하라. 전체를 재생성하지 마라.
- `[수동 추가]` 내용을 삭제하거나 덮어쓰지 마라.
- 최종 문서의 서술은 "시스템이 이렇게 동작한다"는 관점으로 작성하라.
- 코드 근거는 중간 확인용으로만 사용하고, 최종 문서에 넣지 마라.
