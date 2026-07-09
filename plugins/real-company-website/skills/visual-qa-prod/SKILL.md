---
name: visual-qa-prod
description: 프로덕션 빌드를 실브라우저(CDP)로 픽셀 계측하는 시각 QA 절차. UI 변경 후 "실제로 그렇게 보이는가"를 수치로 검증할 때 사용. 마커 grep이나 정적 검사로는 잡히지 않는 CSS 우선순위·뷰포트·타이밍 버그를 잡는다. 코드 수정 없이 판정만.
---

# visual-qa-prod — 프로덕션 실측 시각 QA

**원칙: 인터랙션·모션·레이아웃 QA는 dev 서버가 아니라 prod 빌드로 실측한다.** (근거: dev 모드는 CSP unsafe-eval 부재 시 hydration이 통째로 죽어 모든 인터랙션 검사가 오탐된다 — 실제 사고 사례. 정적 마커 grep은 "CSS가 존재한다"만 보장하고 "적용된다"는 보장하지 않는다.)

## 절차
1. `npm run build` → `npx next start -H 127.0.0.1 -p 3199`(또는 프로젝트 상응 명령). 프로덕션 URL 직접 검사도 가능(배포 직후).
2. 헤드리스 크롬 CDP 연결: chrome-launcher+puppeteer-core/ws(리포 devDependency 재사용, 신규 설치 금지). 렌즈·라운드마다 **디버깅 포트를 다르게**(9345, 9346…) — 병렬 충돌 방지.
3. 모바일은 창 리사이즈가 아니라 **`Emulation.setDeviceMetricsOverride`(390×844, DPR2~3, 터치)** — OS 최소 창폭 때문에 리사이즈는 신뢰 불가(실측 확인된 함정).
4. 검사 후 서버·크롬 반드시 종료, 임시 스크립트 삭제, `git status`로 리포 무변경 확인.

## 계측 레퍼토리 (전부 실전 검증된 방법)
- **겹침**: 두 요소 `getBoundingClientRect()` 교차 면적 — 0이어야 함.
- **오버플로**: `document.documentElement.scrollWidth === innerWidth` (내부 가로 스크롤 레일은 자체 overflow 컨테이너면 무관). 레이아웃 뷰포트 팽창은 fixed 요소 폭이 늘어나는 부수 신호로도 검출.
- **스태거/모션**: `getComputedStyle` transition-delay 계단(0/120/240…) + rAF 40ms 간격 opacity 샘플링으로 **순차 등장을 시간축에서 실측**. `Animation.currentTime` 스크럽+`getComputedTiming()`은 백그라운드 탭에서도 신뢰 가능.
- **CSS 승부**: 기대 스타일이 computed에 실제로 나오는지 — 흔한 패자: transition 단축 속성이 delay 리셋, @layer utilities가 components를 이김, :root에서 var() 간접참조가 굳음, media query 셀렉터 specificity 패배, **서드파티 SDK가 inline style로 position을 덮어써 absolute/inset 크기 지정이 무효화(높이 0 — 네트워크는 전부 200인데 화면은 빈 박스)**. 임베드 위젯은 반드시 computed height>0까지 계측.
- **타이포/정렬**: fontSize·fontWeight 실측, 중앙 정렬은 뷰포트 중심 대비 오프셋(±수 px), eyebrow≠h1 텍스트 대조.
- **터치 타깃**: 인터랙션 요소 rect ≥44px.
- **CLS**: `Page.addScriptToEvaluateOnNewDocument`로 PerformanceObserver(layout-shift, buffered)를 **네비게이션 전에 주입** — 로드 후 주입하면 초기 시프트를 놓친다.
- **reduced-motion**: `emulateMediaFeatures`로 실제 에뮬레이션 후 즉시-완성 상태 계측(transition 0s, opacity 1).
- **이미지**: naturalWidth>0 전수 + Network 도메인 4xx/5xx 감청.
- **verbatim 카피**: innerText 정확 대조(대시·말줄임표 포함) — 오너 확정 문구 검증용.

## 판정 규칙
- 항목별 PASS/FAIL + 실측 수치 + 스크린샷(/tmp/<라운드>-qa/). FAIL은 원인 추정(파일:라인)까지.
- **naturalWidth 함정**: w-디스크립터 srcset에서 naturalWidth는 밀도 보정값(실비트맵÷밀도)이다 — 업스케일 판정은 srcset 후보/최적화 URL의 실제 비트맵 폭으로.
- **네이티브 dialog는 합성 ESC로 안 닫힌다**(KeyboardEvent dispatch 무효) — keyboard.press('Escape') 실입력으로 검증.
- 자동 판정이 애매하면 스크린샷 육안을 정본으로. 셀렉터 실수로 인한 오탐은 재검증 후 정정 표기.
- 알려진 대기 항목(오너 확인 등)은 사전에 제외 목록으로 명시해 소음 제거.
- 풀페이지 캡처(`captureBeyondViewport`)는 sticky 요소 스티칭 아티팩트가 생길 수 있다 — 판정 근거는 뷰포트 단일 캡처로.
