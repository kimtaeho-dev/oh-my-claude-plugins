---
name: website-spec
description: |
  비즈니스 지식 정적 웹사이트(_site/)의 파일 구조, HTML 템플릿, CSS 디자인, JS 동작, 검색 인덱스 형식을 정의.
  biz-init Phase 3 Step D, biz-update 웹사이트 재빌드 시 반드시 이 형식을 따른다.
---

# 정적 웹사이트 형식 정의 (Website Spec)

## 공통 규칙

- 외부 CDN, npm 패키지, 빌드 도구 없이 순수 HTML + CSS + JS만 사용
- CSS는 `assets/style.css` 단일 파일, JS는 `assets/app.js` 단일 파일
- 모든 HTML 요소의 class 이름과 id 형식은 이 문서에 정의된 것을 정확히 사용
- HTML 파일 간 링크는 상대 경로 사용 (예: `../glossary.html`, `domains/auth/policies.html`)
- 앵커 id 형식: 정책 `POL-{NNN}`, 에러 `ERR-{NNN}`, 용어 `term-{한글용어를-kebab-case로}`
- 마크다운 → HTML 변환 시 business-spec 스킬의 형식을 기준으로 매핑

---

## 파일 구조

```
_site/
├── index.html                          # 메인 페이지 (POLICY_CATALOG 기반)
├── glossary.html                       # 전체 용어집 (GLOSSARY.md 기반)
├── domains/
│   └── {domain-name}/
│       ├── policies.html               # 도메인 정책서 (POLICIES.md 기반)
│       ├── spec.html                   # 도메인 기획서 (SPEC.md 기반)
│       └── glossary.html               # 도메인 용어집 (GLOSSARY.md 기반)
├── assets/
│   ├── style.css                       # 전체 스타일
│   └── app.js                          # 검색, 테마, 사이드바 동작
└── search-index.json                   # 클라이언트 사이드 검색 인덱스
```

### 규칙
- `{domain-name}`은 영문 소문자 kebab-case (예: `user-auth`, `payment`)
- 도메인 수만큼 `domains/` 하위 디렉토리 생성
- 위 구조 외 추가 파일을 생성하지 않는다

---

## HTML 공통 템플릿

모든 페이지는 다음 뼈대를 공유한다.

```html
<!DOCTYPE html>
<html lang="ko" data-theme="light">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{페이지 제목} — {프로젝트명} 비즈니스 지식</title>
  <link rel="stylesheet" href="{assets/style.css 상대경로}">
</head>
<body>
  <div class="layout">
    <nav class="sidebar" id="sidebar">
      <div class="sidebar-header">
        <h1 class="sidebar-title">{프로젝트명}</h1>
        <button class="theme-toggle" id="theme-toggle" aria-label="테마 전환">🌙</button>
      </div>
      <div class="search-box">
        <input type="search" class="search-input" id="search-input" placeholder="검색..." aria-label="검색">
        <div class="search-results" id="search-results"></div>
      </div>
      <ul class="nav-list">
        <li class="nav-item">
          <a href="{index.html 상대경로}" class="nav-link {현재 페이지면 active}">정책 카탈로그</a>
        </li>
        <li class="nav-item">
          <a href="{glossary.html 상대경로}" class="nav-link {현재 페이지면 active}">전체 용어집</a>
        </li>
        <!-- 도메인 수만큼 반복 -->
        <li class="nav-item nav-domain">
          <button class="nav-domain-toggle" aria-expanded="false">{도메인 한글명}</button>
          <ul class="nav-sublist">
            <li><a href="{policies.html 상대경로}" class="nav-link {현재 페이지면 active}">정책서</a></li>
            <li><a href="{spec.html 상대경로}" class="nav-link {현재 페이지면 active}">기획서</a></li>
            <li><a href="{glossary.html 상대경로}" class="nav-link {현재 페이지면 active}">용어집</a></li>
          </ul>
        </li>
      </ul>
    </nav>

    <button class="sidebar-toggle" id="sidebar-toggle" aria-label="메뉴 열기">☰</button>

    <main class="content">
      <nav class="breadcrumb" aria-label="현재 위치">
        <!-- 브레드크럼 항목 -->
      </nav>
      <article class="page">
        <!-- 페이지별 콘텐츠 -->
      </article>
    </main>
  </div>
  <script src="{assets/app.js 상대경로}"></script>
</body>
</html>
```

### 브레드크럼 규칙

| 페이지 | 브레드크럼 |
|---|---|
| index.html | `홈` (링크 없음) |
| glossary.html | `홈 > 전체 용어집` |
| domains/{d}/policies.html | `홈 > {도메인 한글명} > 정책서` |
| domains/{d}/spec.html | `홈 > {도메인 한글명} > 기획서` |
| domains/{d}/glossary.html | `홈 > {도메인 한글명} > 용어집` |

- 구분자: `>` (CSS `::before`로 삽입)
- 마지막 항목은 링크 없이 텍스트만 표시
- 형식:

```html
<nav class="breadcrumb" aria-label="현재 위치">
  <a href="{index.html 경로}" class="breadcrumb-item">홈</a>
  <span class="breadcrumb-item">{현재 페이지명}</span>
</nav>
```

---

## 페이지별 콘텐츠 템플릿

### index.html (정책 카탈로그)

POLICY_CATALOG.md를 변환한다.

```html
<article class="page">
  <h1 class="page-title">정책 카탈로그</h1>
  <p class="page-meta">최종 갱신: {YYYY-MM-DD}</p>

  <section class="catalog-summary">
    <h2>요약</h2>
    <table class="summary-table">
      <thead>
        <tr><th>도메인</th><th>정책 수</th><th>에러 시나리오 수</th></tr>
      </thead>
      <tbody>
        <tr><td><a href="domains/{domain}/policies.html">{도메인 한글명}</a></td><td>{N}</td><td>{M}</td></tr>
        <!-- 도메인 수만큼 반복 -->
        <tr class="summary-total"><td>합계</td><td>{총N}</td><td>{총M}</td></tr>
      </tbody>
    </table>
  </section>

  <!-- 도메인 수만큼 반복 -->
  <section class="catalog-domain" id="{domain-name}">
    <h2>{도메인 한글명}</h2>
    <table class="policy-index-table">
      <thead>
        <tr><th>정책 ID</th><th>규칙 요약</th><th>관련 기능</th><th>분류</th></tr>
      </thead>
      <tbody>
        <tr>
          <td><a href="domains/{domain}/policies.html#POL-{NNN}" class="xref">POL-{NNN}</a></td>
          <td>{규칙 한 줄 요약}</td>
          <td>{기능 ID}</td>
          <td><span class="tag tag-{분류영문}">{분류}</span></td>
        </tr>
      </tbody>
    </table>
  </section>

  <section class="catalog-common" id="common-policies">
    <h2>공통 정책</h2>
    <table class="policy-index-table">
      <thead>
        <tr><th>정책 ID</th><th>규칙 요약</th><th>적용 도메인</th><th>비고</th></tr>
      </thead>
      <tbody>
        <tr>
          <td>COMMON-{NNN}</td>
          <td>{규칙 한 줄 요약}</td>
          <td>{도메인 목록}</td>
          <td>{비고}</td>
        </tr>
      </tbody>
    </table>
  </section>
</article>
```

### domains/{domain}/policies.html (도메인 정책서)

POLICIES.md를 변환한다.

```html
<article class="page">
  <h1 class="page-title">{도메인 한글명} 도메인 — 서비스 정책서</h1>
  <p class="page-meta">최종 갱신: {YYYY-MM-DD}</p>

  <!-- 기능 수만큼 반복 -->
  <section class="feature-section" id="{app}.{기능_ID}">
    <h2>{app}.{기능_ID}</h2>

    <h3>Validation 규칙</h3>
    <!-- 정책 수만큼 반복 -->
    <section class="policy" id="POL-{NNN}">
      <h4>
        <span class="policy-id">POL-{NNN}</span>
        <span class="policy-title">{규칙 제목}</span>
        <span class="tag tag-validation">validation</span>
      </h4>
      <dl class="policy-fields">
        <dt>규칙</dt><dd>{비즈니스 언어로 규칙 설명}</dd>
        <dt>위반 시</dt><dd>{사용자에게 어떤 일이 일어나는가}</dd>
        <dt>비즈니스 배경</dt><dd>{왜 이 규칙이 필요한가}</dd>
      </dl>
    </section>

    <h3>상태 정책</h3>
    <section class="policy" id="POL-{NNN}">
      <h4>
        <span class="policy-id">POL-{NNN}</span>
        <span class="policy-title">{상태 전이 규칙 제목}</span>
        <span class="tag tag-state">상태 전이</span>
      </h4>
      <dl class="policy-fields">
        <dt>규칙</dt><dd>{설명}</dd>
        <dt>전이</dt><dd><span class="state-from">{상태 A}</span> → <span class="state-to">{상태 B}</span></dd>
        <dt>전이 조건</dt><dd>{조건}</dd>
        <dt>역전이</dt><dd>{가능/불가}</dd>
        <dt>영향</dt><dd>{변화}</dd>
      </dl>
    </section>

    <h3>설정값</h3>
    <section class="policy" id="POL-{NNN}">
      <h4>
        <span class="policy-id">POL-{NNN}</span>
        <span class="policy-title">{설정 제목}</span>
        <span class="tag tag-config">설정값</span>
      </h4>
      <dl class="policy-fields">
        <dt>규칙</dt><dd>{설명}</dd>
        <dt>비즈니스 의미</dt><dd>{영향}</dd>
        <dt>변경 시 영향</dt><dd>{변화}</dd>
      </dl>
    </section>

    <h3>에러 시나리오</h3>
    <section class="policy error-scenario" id="ERR-{NNN}">
      <h4>
        <span class="policy-id">ERR-{NNN}</span>
        <span class="policy-title">{에러 제목}</span>
      </h4>
      <dl class="policy-fields">
        <dt>메시지</dt><dd class="error-message">"{실제 에러 메시지}"</dd>
        <dt>발생 상황</dt><dd>{상황 설명}</dd>
        <dt>사용자 영향</dt><dd>{영향}</dd>
        <dt>대응 방법</dt><dd>{대응 방법}</dd>
      </dl>
    </section>
  </section>
</article>
```

### domains/{domain}/spec.html (도메인 기획서)

SPEC.md를 변환한다.

```html
<article class="page">
  <h1 class="page-title">{도메인 한글명} 도메인 — 기획서</h1>
  <p class="page-meta">최종 갱신: {YYYY-MM-DD}</p>

  <section class="domain-overview">
    <h2>도메인 개요</h2>
    <p>{도메인 역할 설명}</p>
  </section>

  <!-- 기능 수만큼 반복 -->
  <section class="spec-feature" id="spec-{기능_slug}">
    <h2>{기능 한글명}</h2>

    <h3>이 기능은 무엇인가</h3>
    <p>{기능 설명}</p>

    <h3>사용자 시나리오</h3>
    <ol class="scenario-steps">
      <li>{단계 설명}</li>
      <!-- 단계 수만큼 반복 -->
    </ol>

    <h3>왜 이렇게 동작하는가</h3>
    <ul class="design-decisions">
      <li><strong>{설계 결정}</strong>: {이유}</li>
    </ul>

    <h3>주요 정책</h3>
    <ul class="related-policies">
      <li><a href="policies.html#POL-{NNN}" class="xref">POL-{NNN}</a>: {정책 한 줄 요약}</li>
    </ul>

    <h3>에러 상황</h3>
    <ul class="related-errors">
      <li><a href="policies.html#ERR-{NNN}" class="xref">ERR-{NNN}</a>: {에러 요약} — {대응 방법}</li>
    </ul>

    <h3>관련 용어</h3>
    <ul class="related-terms">
      <li><a href="glossary.html#term-{slug}" class="xref">{용어}</a>: {간략 정의}</li>
    </ul>
  </section>
</article>
```

### glossary.html (전체 용어집)

전체 GLOSSARY.md를 변환한다.

```html
<article class="page">
  <h1 class="page-title">전체 용어집</h1>
  <p class="page-meta">최종 갱신: {YYYY-MM-DD}</p>

  <nav class="alpha-nav" aria-label="가나다 색인">
    <a href="#section-ㄱ">ㄱ</a>
    <a href="#section-ㄴ">ㄴ</a>
    <!-- 실제 존재하는 초성만 -->
  </nav>

  <!-- 초성 그룹별 -->
  <section class="glossary-group" id="section-{초성}">
    <h2>{초성}</h2>
    <!-- 용어 수만큼 반복 -->
    <div class="term-entry" id="term-{slug}">
      <h3 class="term-name">{한글 용어}</h3>
      <dl class="term-fields">
        <dt>정의</dt><dd>{이 프로젝트에서의 의미}</dd>
        <dt>소속 도메인</dt><dd>{도메인 한글명}</dd>
        <dt>혼동 주의</dt><dd>{다른 용어와의 차이}</dd>
      </dl>
    </div>
  </section>

  <section class="glossary-conflicts" id="definition-conflicts">
    <h2>정의 불일치 목록</h2>
    <table class="summary-table">
      <thead>
        <tr><th>용어</th><th>도메인 A 정의</th><th>도메인 B 정의</th><th>상태</th></tr>
      </thead>
      <tbody>
        <tr>
          <td>{용어}</td><td>{정의 A}</td><td>{정의 B}</td><td>{미해소 / 해소}</td>
        </tr>
      </tbody>
    </table>
  </section>
</article>
```

### domains/{domain}/glossary.html (도메인 용어집)

도메인 GLOSSARY.md를 변환한다.

```html
<article class="page">
  <h1 class="page-title">{도메인 한글명} 도메인 — 용어집</h1>
  <p class="page-meta">최종 갱신: {YYYY-MM-DD}</p>

  <!-- 용어 수만큼 반복 -->
  <div class="term-entry" id="term-{slug}">
    <h3 class="term-name">{한글 용어}</h3>
    <dl class="term-fields">
      <dt>정의</dt><dd>{의미}</dd>
      <dt>혼동 주의</dt><dd>{차이 — 해당 시만}</dd>
      <dt>예시</dt><dd>{예시 — 해당 시만}</dd>
    </dl>
  </div>
</article>
```

### 규칙
- 해당 섹션이 소스 마크다운에 없으면 해당 `<section>` 또는 `<h3>` 블록 자체를 생략
- 선택적 필드 (`비즈니스 배경`, `혼동 주의`, `예시` 등)는 소스에 내용이 있을 때만 `<dt>`/`<dd>` 쌍을 출력
- `[수동 추가]` 태그는 HTML에서 `<span class="manual-tag">[수동 추가]</span>`로 변환

---

## CSS 디자인 시스템

### 색상 토큰

```css
:root {
  /* 배경 */
  --color-bg: #ffffff;
  --color-surface: #f8f9fa;
  --color-border: #dee2e6;

  /* 텍스트 */
  --color-text-primary: #212529;
  --color-text-secondary: #495057;
  --color-text-muted: #868e96;

  /* 강조 */
  --color-accent: #228be6;
  --color-accent-hover: #1971c2;

  /* 사이드바 */
  --color-sidebar-bg: #f1f3f5;
  --color-sidebar-active: #e7f5ff;

  /* 분류 태그 */
  --color-tag-validation: #d0ebff;
  --color-tag-validation-text: #1864ab;
  --color-tag-state: #d3f9d8;
  --color-tag-state-text: #2b8a3e;
  --color-tag-config: #fff3bf;
  --color-tag-config-text: #e67700;
  --color-tag-permission: #e8d0f0;
  --color-tag-permission-text: #7b2d8e;
  --color-tag-constraint: #ffe0e0;
  --color-tag-constraint-text: #c92a2a;
}

[data-theme="dark"] {
  --color-bg: #1a1b1e;
  --color-surface: #25262b;
  --color-border: #373a40;

  --color-text-primary: #c1c2c5;
  --color-text-secondary: #909296;
  --color-text-muted: #5c5f66;

  --color-accent: #4dabf7;
  --color-accent-hover: #74c0fc;

  --color-sidebar-bg: #1e1f23;
  --color-sidebar-active: #1b2838;

  --color-tag-validation: #1b2838;
  --color-tag-validation-text: #74c0fc;
  --color-tag-state: #1b3026;
  --color-tag-state-text: #69db7c;
  --color-tag-config: #302b1a;
  --color-tag-config-text: #ffd43b;
  --color-tag-permission: #2b1a33;
  --color-tag-permission-text: #cc5de8;
  --color-tag-constraint: #331a1a;
  --color-tag-constraint-text: #ff6b6b;
}
```

### 타이포그래피

```css
:root {
  --font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Noto Sans KR", sans-serif;
  --font-mono: "SF Mono", Consolas, "Noto Sans Mono", monospace;

  --font-size-xs: 0.75rem;    /* 12px */
  --font-size-sm: 0.875rem;   /* 14px */
  --font-size-base: 1rem;     /* 16px */
  --font-size-lg: 1.125rem;   /* 18px */
  --font-size-xl: 1.5rem;     /* 24px */
  --font-size-2xl: 2rem;      /* 32px */

  --line-height-tight: 1.25;
  --line-height-normal: 1.6;
  --line-height-relaxed: 1.8;
}
```

### 헤딩 크기

| 요소 | font-size | font-weight | margin-top |
|---|---|---|---|
| h1 (`.page-title`) | `--font-size-2xl` | 700 | 0 |
| h2 | `--font-size-xl` | 600 | `--space-8` |
| h3 | `--font-size-lg` | 600 | `--space-6` |
| h4 (정책 제목) | `--font-size-base` | 600 | `--space-4` |

### 스페이싱

```css
:root {
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-3: 0.75rem;   /* 12px */
  --space-4: 1rem;      /* 16px */
  --space-5: 1.5rem;    /* 24px */
  --space-6: 2rem;      /* 32px */
  --space-7: 2.5rem;    /* 40px */
  --space-8: 3rem;      /* 48px */
}
```

### 레이아웃

```css
:root {
  --sidebar-width: 260px;
  --content-max-width: 800px;
  --border-radius: 4px;
}
```

### 핵심 레이아웃 CSS

```css
.layout {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  width: var(--sidebar-width);
  background: var(--color-sidebar-bg);
  border-right: 1px solid var(--color-border);
  padding: var(--space-4);
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  overflow-y: auto;
}

.content {
  margin-left: var(--sidebar-width);
  padding: var(--space-6);
  max-width: var(--content-max-width);
  width: 100%;
}
```

### 분류 태그 CSS

```css
.tag {
  display: inline-block;
  padding: var(--space-1) var(--space-2);
  border-radius: var(--border-radius);
  font-size: var(--font-size-xs);
  font-weight: 500;
}

.tag-validation { background: var(--color-tag-validation); color: var(--color-tag-validation-text); }
.tag-state { background: var(--color-tag-state); color: var(--color-tag-state-text); }
.tag-config { background: var(--color-tag-config); color: var(--color-tag-config-text); }
.tag-permission { background: var(--color-tag-permission); color: var(--color-tag-permission-text); }
.tag-constraint { background: var(--color-tag-constraint); color: var(--color-tag-constraint-text); }
```

### 분류명 → 태그 class 매핑

| 분류 (한글) | class |
|---|---|
| validation | `tag-validation` |
| 상태 전이 | `tag-state` |
| 설정값 | `tag-config` |
| 권한 | `tag-permission` |
| 데이터 제약 | `tag-constraint` |

---

## 컴포넌트 명세

### 사이드바

- 최상단: 프로젝트명 (`sidebar-title`) + 테마 토글 버튼
- 그 아래: 검색 입력 필드
- 네비게이션 목록: 정책 카탈로그, 전체 용어집은 최상위 항목. 도메인은 접기/펼치기 가능한 그룹
- 현재 페이지 링크에 `active` class 추가
- 도메인 그룹: `nav-domain-toggle` 버튼 클릭 시 `nav-sublist`의 표시/숨김 토글. `aria-expanded` 속성 갱신

### 검색 결과

```html
<div class="search-results" id="search-results">
  <!-- 결과가 있을 때만 표시 -->
  <a class="search-result-item" href="{url}">
    <span class="search-result-title">{제목}</span>
    <span class="search-result-excerpt">{본문 발췌 — 매칭 부분 <mark>하이라이트</mark>}</span>
    <span class="search-result-meta">{type} · {도메인 한글명}</span>
  </a>
</div>
```

### 정책 카드

```css
.policy {
  border: 1px solid var(--color-border);
  border-radius: var(--border-radius);
  padding: var(--space-4);
  margin-top: var(--space-4);
  background: var(--color-surface);
}

.policy h4 {
  display: flex;
  align-items: center;
  gap: var(--space-2);
  margin: 0 0 var(--space-3) 0;
}

.policy-id {
  font-family: var(--font-mono);
  font-size: var(--font-size-sm);
  color: var(--color-accent);
}

.policy-fields dt {
  font-weight: 600;
  color: var(--color-text-secondary);
  margin-top: var(--space-2);
}

.policy-fields dd {
  margin-left: 0;
  margin-top: var(--space-1);
}
```

### 테이블

```css
.summary-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: var(--space-4);
}

.summary-table th,
.summary-table td {
  padding: var(--space-2) var(--space-3);
  border: 1px solid var(--color-border);
  text-align: left;
}

.summary-table th {
  background: var(--color-surface);
  font-weight: 600;
}

.summary-table tbody tr:nth-child(even) {
  background: var(--color-surface);
}

.summary-total td {
  font-weight: 700;
  border-top: 2px solid var(--color-border);
}
```

### 크로스 레퍼런스 링크

```css
.xref {
  color: var(--color-accent);
  text-decoration: none;
  border-bottom: 1px dashed var(--color-accent);
}

.xref:hover {
  color: var(--color-accent-hover);
  border-bottom-style: solid;
}
```

---

## JavaScript 동작 명세

### 테마 전환

```javascript
// 초기화: 페이지 로드 시
const saved = localStorage.getItem('theme');
const preferred = window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
document.documentElement.dataset.theme = saved || preferred;

// 토글 버튼 클릭 시
function toggleTheme() {
  const current = document.documentElement.dataset.theme;
  const next = current === 'dark' ? 'light' : 'dark';
  document.documentElement.dataset.theme = next;
  localStorage.setItem('theme', next);
  // 버튼 텍스트: light → '🌙', dark → '☀️'
}
```

### 검색

```javascript
// 페이지 로드 시 search-index.json 로드
let searchIndex = [];
fetch('{assets 상대경로}/search-index.json')
  .then(r => r.json())
  .then(data => { searchIndex = data.entries; });

// 입력 이벤트 (200ms debounce)
function handleSearch(query) {
  if (query.length < 2) { hideResults(); return; }
  const terms = query.toLowerCase().split(/\s+/);
  const results = searchIndex
    .filter(entry =>
      terms.every(t =>
        entry.title.toLowerCase().includes(t) ||
        entry.content.toLowerCase().includes(t)
      )
    )
    .slice(0, 10);
  renderResults(results, query);
}

// 결과 렌더링: search-result-item 구조로 출력
// 매칭 텍스트를 <mark>로 감싸기
// 결과 없으면 "검색 결과가 없습니다" 표시
```

### 모바일 사이드바

```javascript
// sidebar-toggle 클릭 시
function toggleSidebar() {
  document.body.classList.toggle('sidebar-open');
}

// 콘텐츠 영역 클릭 시 사이드바 닫기
document.querySelector('.content').addEventListener('click', () => {
  document.body.classList.remove('sidebar-open');
});
```

### 도메인 접기/펼치기

```javascript
// nav-domain-toggle 클릭 시
function toggleDomainNav(button) {
  const expanded = button.getAttribute('aria-expanded') === 'true';
  button.setAttribute('aria-expanded', !expanded);
  button.nextElementSibling.style.display = expanded ? 'none' : 'block';
}
```

---

## search-index.json 형식

```json
{
  "version": 1,
  "generatedAt": "{YYYY-MM-DD}",
  "entries": [
    {
      "id": "{type}-{domain}-{section-id}",
      "type": "policy | spec | glossary | catalog",
      "domain": "{domain-name | _global}",
      "title": "{섹션 제목 — 한글}",
      "content": "{본문 텍스트. HTML 태그 제거. 최대 200자}",
      "url": "{상대 경로}#{앵커 id}",
      "tags": ["{분류1}"]
    }
  ]
}
```

### id 생성 규칙

| type | id 형식 | 예시 |
|---|---|---|
| policy | `policy-{domain}-POL-{NNN}` | `policy-auth-POL-001` |
| spec | `spec-{domain}-{기능_slug}` | `spec-auth-login` |
| glossary | `glossary-{domain}-{용어_slug}` | `glossary-auth-세션` |
| catalog | `catalog-{domain}` | `catalog-auth` |

### 규칙
- 모든 정책 (`POL-*`, `ERR-*`), 기획서 기능, 용어 각각을 하나의 entry로 등록
- `content`는 본문에서 HTML 태그를 제거하고 공백을 정규화한 후 200자까지 자른다
- `url`의 상대 경로는 `_site/` 기준 (예: `domains/auth/policies.html#POL-001`)
- `tags`에는 분류(validation, 상태 전이 등)를 배열로 포함

---

## 반응형 디자인

### 브레이크포인트

| 범위 | 동작 |
|---|---|
| `>= 1024px` | 사이드바 고정 표시, 콘텐츠 옆에 배치 |
| `768px ~ 1023px` | 사이드바 접기 가능, 닫히면 콘텐츠 전폭 |
| `< 768px` | 사이드바 숨김, 햄버거 토글로 오버레이 표시 |

### 반응형 CSS

```css
.sidebar-toggle {
  display: none;
}

@media (max-width: 1023px) {
  .sidebar {
    transform: translateX(-100%);
    transition: transform 0.2s ease;
    z-index: 100;
  }

  .sidebar-open .sidebar {
    transform: translateX(0);
  }

  .sidebar-toggle {
    display: block;
    position: fixed;
    top: var(--space-3);
    left: var(--space-3);
    z-index: 50;
    background: var(--color-surface);
    border: 1px solid var(--color-border);
    border-radius: var(--border-radius);
    padding: var(--space-2);
    cursor: pointer;
  }

  .content {
    margin-left: 0;
  }
}
```

---

## 인쇄 스타일

```css
@media print {
  .sidebar,
  .sidebar-toggle,
  .search-box,
  .theme-toggle,
  .alpha-nav {
    display: none !important;
  }

  .content {
    margin-left: 0;
    max-width: 100%;
    padding: 0;
  }

  .layout {
    display: block;
  }

  body {
    color: #000;
    background: #fff;
  }

  a.xref::after {
    content: " (" attr(href) ")";
    font-size: var(--font-size-xs);
    color: #666;
  }

  h2 {
    page-break-before: auto;
    page-break-after: avoid;
  }

  .policy, .term-entry {
    page-break-inside: avoid;
  }
}
```
