$ARGUMENTS 도메인에 변경이 발생했다.
코드 변경 사항을 분석하여 코드 지식 문서를 갱신하라.

## 인자 형식

`{도메인명}` — 변경이 발생한 도메인 (예: poker, room, jira)
인자가 없으면 git diff로 변경된 파일을 분석하여 영향받은 도메인을 자동 판별하라.

## 갱신 절차

### 1. 변경 범위 파악

1. git diff (또는 사용자가 설명한 변경 내용)에서 변경된 파일 목록을 확인하라
2. `.claude/docs/CODE_STRUCTURE.md`에서 변경된 파일이 속한 도메인을 식별하라
3. `.claude/docs/domains/{domain}/FEATURES.md`에서 영향받는 기능 ID를 특정하라

### 2. 도메인 문서 갱신

변경된 기능에 대해:

1. `.claude/docs/domains/{domain-name}/FEATURES.md`
   - 변경된 기능의 코드 위치 업데이트
   - impacts/affected_by 관계 변경 여부 확인 및 갱신
   - 변경 시 체크리스트 갱신
2. `.claude/docs/domains/{domain-name}/ENTITIES.md`
   - 엔티티 필드/상태 전이 변경 시 갱신
3. `.claude/docs/domains/{domain-name}/OVERVIEW.md`
   - 도메인 역할이나 앱 구성 변경 시 갱신

### 3. 전체 문서 갱신

1. `.claude/docs/DOMAIN_MAP.md` — 해당 도메인 행의 영향 관계 테이블 갱신
2. `.claude/docs/DEPENDENCY_MATRIX.md` — 새로운 교차 도메인 의존이 생겼다면 추가
3. `.claude/docs/SERVICE_FLOW.md` — 흐름에 영향이 있으면 갱신
4. `.claude/docs/domains/{domain-name}/CHANGE_LOG.md` — 변경 이력 추가

### 4. 교차 검증

- [ ] DOMAIN_MAP.md의 기능 ID와 FEATURES.md의 기능 ID가 일치하는가
- [ ] 변경된 impacts에 대응하는 affected_by가 상대 도메인에 존재하는가
- [ ] 불일치가 있으면 수정하라

## 규칙

- 변경된 기능 ID에 해당하는 부분만 갱신하라. 전체를 재생성하지 마라.
- 기존 문서의 구조와 스타일을 유지하라.
- 코드에서 확인되지 않는 관계는 `[미확인]`으로 표기하라.
