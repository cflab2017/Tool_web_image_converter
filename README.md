# 이미지 포맷 변환기 (Image Format Converter)

이미지를 **PNG · JPEG · WebP** 로 상호 변환하고 다운로드하는 단일 HTML 도구입니다.
빌드 도구·서버·프레임워크 없이 `index.html` 파일 하나로 완결됩니다.

> 🔒 모든 변환은 **브라우저 안에서만** 처리됩니다. 파일이 서버로 업로드되지 않습니다.

▶ **데모 (자막 포함):** [한국어 `docs/demo_kr.html`](docs/demo_kr.html) · [English `docs/demo_en.html`](docs/demo_en.html) — 드롭 → 포맷 선택 → 품질 → 배경 지우기 → 다운로드 워크스루를 자막과 함께 자동 재생합니다. 자막은 [`demo_kr.vtt`](docs/demo_kr.vtt) / [`demo_en.vtt`](docs/demo_en.vtt)(WebVTT)로도 제공됩니다.

📖 **사용법 글:** [`docs/usage-guide.md`](docs/usage-guide.md) — 사용법 중심으로 정리한 글(블로그·홈페이지 게시용).

## 기능

- 이미지 1장 입력 — 드래그&드롭 또는 클릭해 선택 (PNG·JPG·WebP·GIF·BMP·SVG 등 브라우저가 디코드 가능한 모든 포맷)
- 출력 포맷 선택: **PNG / JPEG / WebP**
- 손실 포맷(JPEG·WebP)용 품질 슬라이더 (0–100)
- JPEG 변환 시 투명 영역을 채울 배경색 지정 (기본 흰색) — 투명 영역이 검게 나오는 문제 방지
- 미리보기(체커보드 배경으로 투명도 표시), 해상도·원본/변환 용량·증감률(%) 표시
- 확장자만 교체한 가짜 변환이 아닌 **실제 재인코딩** (`canvas.toBlob`의 MIME으로 인코딩 결정)

## 사용법

`index.html` 을 브라우저에서 열면 됩니다. 설치·서버 불필요.

```
git clone https://github.com/cflab2017/Tool_web_image_converter.git
cd Tool_web_image_converter
# index.html 을 브라우저로 열기 (더블클릭 또는)
xdg-open index.html   # Linux
open index.html       # macOS
```

1. 이미지를 드롭존에 끌어다 놓거나 클릭해 선택
2. 출력 포맷(PNG/JPEG/WebP) 선택 — JPEG·WebP는 품질 슬라이더, JPEG는 배경색 지정 가능
3. **다운로드** 버튼으로 저장 (파일명은 원본 확장자만 교체)

## 동작 방식

브라우저 내장 **Canvas API** 만 사용합니다 (외부 라이브러리 없음).

- `createImageBitmap(file, { imageOrientation: 'from-image' })` 으로 EXIF 회전을 반영해 디코드
- `<canvas>` 에 그린 뒤 `canvas.toBlob(callback, type, quality)` 로 재인코딩
- JPEG는 알파 채널이 없어, 그리기 전에 배경색으로 `fillRect` 후 `drawImage`
- 결과 Blob을 `URL.createObjectURL` 로 미리보기·다운로드에 연결하고 사용 후 `revokeObjectURL` 로 정리

## 기술 스택

- 순수 HTML / CSS / JavaScript (인라인, 단일 파일)
- 외부 의존성 없음 — Google Fonts(JetBrains Mono, Gowun Dodum)만 사용
- 다크 "앰버 CRT 터미널" 테마

## 라이선스 / 후원

☕ 후원: 도구가 마음에 들면 후원으로 응원해 주세요. (`index.html` 의 `DONATE_URL` 상수)
