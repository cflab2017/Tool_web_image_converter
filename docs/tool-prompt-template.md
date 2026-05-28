# 단일 HTML 브라우저 도구 제작 — 재사용 프롬프트 템플릿

> 이번 "이미지 포맷 변환기" 작업에서 확립한 규약을 일반화한 템플릿입니다.
> 비슷한 도구를 만들 때 이 문서를 Claude Code(또는 다른 AI)에 붙여넣고, **[채울 항목]** 부분만 바꿔 지시하세요.

---

## 0. 사용법

1. 아래 **"1. 이번에 만들 도구"** 의 대괄호 항목을 채운다.
2. 전체를 그대로 복사해 작업을 지시한다.
3. "먼저 계획을 요약하고, 단일 HTML로 구현한 뒤 검증 기준을 스스로 확인하라"고 덧붙인다.

---

## 1. 이번에 만들 도구 (채울 항목)

- **도구 이름**: [예: 이미지 포맷 변환기]
- **한 줄 목적**: [예: 이미지를 PNG·JPEG·WebP로 상호 변환해 다운로드]
- **입력**: [예: 이미지 파일 1장 (드래그&드롭 또는 클릭 선택)]
- **출력**: [예: 선택한 포맷으로 변환된 파일 다운로드]
- **핵심 동작 한 가지**: [예: "이미지를 넣으면 원하는 포맷으로 받는다"]
- **꼭 필요한 옵션(최소)**: [예: 출력 포맷 3종, 품질 슬라이더]
- **사용할 브라우저 API**: [예: Canvas `toBlob`, `createImageBitmap`]

---

## 2. 변하지 않는 핵심 원칙

- **단일 `.html` 파일**로 완결한다 (CSS·JS 인라인). 빌드 도구·서버·프레임워크 없음.
- **모든 처리는 브라우저 안에서.** 파일이 서버로 업로드되지 않는다(프라이버시). 외부 네트워크 의존 없음.
- **외부 라이브러리 금지.** 브라우저 내장 API만 사용. 예외: Google Fonts(폰트만).
- **옵션은 최소화.** 핵심 동작 하나를 흠 없이. 부가 기능(일괄 처리, 편집, 고급 옵션 등)은 1차 버전에서 제외.
- **진짜 동작이어야 한다.** 겉보기만 바꾼 가짜 금지(예: 변환은 실제 재인코딩이어야 함).
- **경계에서만 방어.** 사용자 입력·디코드 실패 등 외부 경계는 친절한 에러 메시지로 처리. 내부 로직엔 불필요한 방어 코드 금지.

---

## 3. 디자인 시스템 — 다크 "앰버 CRT 터미널" 테마

CODINGNOW 도구들과 통일. 아래 토큰을 그대로 사용한다.

```css
:root{
  --bg:#0b0a08; --panel:#141210; --panel-2:#1c1916;
  --amber:#ffb000; --amber-dim:#b8801f; --line:#2a251e;
  --muted:#7a6f5c; --text:#f3e7cf;
  --danger:#ff5f56; --ok:#7ec96b;
}
```

- **폰트**: `JetBrains Mono`(UI) + `Gowun Dodum`(한글) — Google Fonts.
  ```html
  <link href="https://fonts.googleapis.com/css2?family=Gowun+Dodum&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet" />
  ```
- **CRT 디테일**: 스캔라인 오버레이(`body::before` repeating-linear-gradient), 비네팅(`body::after` radial-gradient), 제목 옆 깜빡이는 커서.
- **강조**: 활성 요소·버튼·포커스는 `--amber`. hover/dragover 시 앰버로 강조.
- **레이아웃**: 좌우 2단 그리드, 모바일에서 1단으로 쌓임(`@media (max-width:720~820px)`).
- **헤더**: `브랜드(CODINGNOW · …)` → `h1 제목 + 커서` → `부제`.
- **푸터**: 한 줄 설명 + `☕ 후원하기` 링크.
- 투명도를 다루는 도구라면 미리보기 배경에 **체커보드 패턴**을 깔아 투명 영역을 드러낸다.

---

## 4. 반드시 포함하는 공통 규약

### 4-1. 후원 링크
- 상수로 분리: `const DONATE_URL = "https://paypal.me/codingnow";`
- 푸터에 `☕ 후원하기` 링크로 연결.

### 4-2. SEO / OG 메타태그
- `description`, `og:type`, `og:title`, `og:description` 포함. `lang`, `viewport`, `charset` 기본.

### 4-3. KR/EN 언어 전환 (인페이지 i18n)
페이지 새로고침 없이 텍스트만 즉시 전환. 작업 중 상태(입력값 등) 유지. 선택은 `localStorage`에 저장.

- 정적 텍스트에 `data-i18n="키"`(필요 시 `data-i18n-title="키"`)를 붙인다. 괄호 주석 등은 별도 `<span>`으로 분리.
- 동적 문자열(에러 메시지 등)은 사전에서 `I18N[currentLang].키`로 꺼내 쓴다.
- 헤더 우측 상단에 `[KR][EN]` 세그먼트 토글.

```js
const I18N = { ko: { /* 키: 값 */ }, en: { /* 키: 값 */ } };
let currentLang = "ko";
function applyLang(lang){
  if (!I18N[lang]) lang = "ko";
  currentLang = lang;
  const dict = I18N[lang];
  document.documentElement.lang = lang;
  document.title = dict.docTitle;
  document.querySelectorAll("[data-i18n]").forEach(el => { const v = dict[el.dataset.i18n]; if (v != null) el.textContent = v; });
  document.querySelectorAll("[data-i18n-title]").forEach(el => { const v = dict[el.dataset.i18nTitle]; if (v != null) el.title = v; });
  [...langToggle.children].forEach(b => b.classList.toggle("active", b.dataset.lang === lang));
  try { localStorage.setItem("tool_lang", lang); } catch(e){}
}
// 초기화: localStorage 값으로 applyLang() 호출, 토글 클릭 시 applyLang(btn.dataset.lang)
```

### 4-4. 에러 처리
- 비정상 입력·디코드 실패 등은 화면에 친절한 메시지(`⚠ …`)로 표시하고 조용히 복구.

---

## 5. 산출물 체크리스트

1. **`index.html`** — 도구 본체 (위 원칙·테마·규약 적용).
2. **데모(선택)** — `docs/demo_kr.html`, `docs/demo_en.html`: 자동 재생 워크스루.
   - 목업 UI 위에서 **가짜 커서**가 이동하며 사용 흐름을 시연.
   - 자막은 **인라인 WebVTT 큐**로 구동되는 하단 오버레이로 표시(타임라인과 동기). 같은 자막을 `docs/demo_*.vtt`로도 저장.
   - 컨트롤: 다시 재생 / 일시정지 / 자막 ON·OFF(CC) / KR·EN 전환.
3. **사용법 글(선택)** — `docs/usage-guide.md`: 사용법 중심, FAQ 포함, 상단에 스크린샷.
4. **스크린샷(선택)** — `docs/images/`에 저장하고 글에 삽입.

---

## 6. 검증 (스스로 확인)

- **JS 문법 검사**: 인라인 스크립트를 추출해 `node --check`.
- **헤드리스 브라우저(Playwright)로 실제 동작 검증**:
  - 핵심 동작이 진짜인지(예: 출력 파일의 매직넘버/헤더 확인).
  - 옵션 변경이 결과에 반영되는지, 경계 조건(빈 입력 등)에서 UI가 올바른지.
  - 스크린샷으로 외형 확인.
- **반응형**: 데스크탑·모바일 레이아웃 점검.
- 완료 전, 도구별 "완료 검증 기준"을 항목별로 직접 통과시킨다.

> 참고: 격리된 샌드박스에서는 Google Fonts(CDN)가 차단될 수 있어 캡처 시 폰트가 대체될 수 있다. 기능 검증에는 영향 없음.

---

## 7. 작업 워크플로우

1. 지정 **개발 브랜치**에서 작업 (없으면 생성).
2. 명확한 메시지로 **커밋**.
3. 변경을 브랜치에 **푸시**.
4. 요청 시 **PR 생성 → main 머지**. (PR은 사용자가 요청할 때만 생성)
5. 위험/비가역 작업(강제 푸시, 삭제 등)은 사전 확인.

---

## 8. 한 줄 요약

> "[핵심 동작]" 하나를, 외부 의존 없이 단일 HTML로, 업로드 없이 브라우저에서, 앰버 CRT 테마로, 최소 옵션으로 흠 없이.
