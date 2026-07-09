# real-company-website — AI스러운 웹사이트를 진짜 회사 웹사이트로

AI(ChatGPT·Claude·v0·Lovable…)가 만든 웹사이트는 서로 닮는다: Inter 폰트, 보라-인디고 그라디언트, 아이콘 3카드, "미래를 만듭니다"류 헤드라인, 화면이 스스로를 설명하는 부제. 이 리포는 그 **AI스러움을 판별하고 교정해 실제 회사 웹사이트처럼 만드는** Claude Code 스킬 6종과 플레이북을 담는다.

실제 한국 B2B 제조사 웹사이트를 수십 라운드에 걸쳐 교정하며 실증한 절차 + 외부 리서치(AI slop 판별 비평, 카피라이팅 원칙, 한국 기업 사이트 관습, 디자인 토큰 오픈소스)를 근거로 한다.

## 스킬 6종

| 스킬 | 역할 |
|---|---|
| **`/de-ai-website`** | AI스러움 판별 체크리스트(시각·카피·구성) → 교정 절차 → 재발 방지 게이트 |
| **`/benchmark-design`** | 벤치마크 사이트·녹화 영상에서 디자인 문법(설명 방식·그리드·모션·색·타이포)을 실측 추출 → 적용 매핑표 |
| **`/visual-qa-prod`** | 프로덕션 빌드를 실브라우저(CDP)로 픽셀 계측 — CSS 우선순위·뷰포트·타이밍 버그를 수치로 검증 |
| **`/design-quality-loop`** | 스캔(적대검증)→수정(모델 분담)→실측→도달 판정을 "품질 도달"까지 반복하는 표준 루프 |
| **`/real-asset-pipeline`** | 실물 사진·영상·엑셀 자산의 보존→판독→변환→**버저닝**→게재 게이트 (캐시 스테일 사고 방지) |
| **`/multi-model-routing`** | 투 메인 멀티모델 라우팅 — 판단/실행 분리, 로컬 프로브 검증 절차, 교차 검증 규칙 |

## 설치

**방법 1 — 스킬 디렉터리에 복사(가장 단순):**
```bash
git clone https://github.com/minseolee0830-stack/real-company-website.git
cp -R real-company-website/plugins/real-company-website/skills/* ~/.claude/skills/
```
이후 Claude Code에서 `/de-ai-website`, `/benchmark-design`, `/visual-qa-prod`로 호출.

**방법 2 — 플러그인 마켓플레이스:**
```
/plugin marketplace add minseolee0830-stack/real-company-website
/plugin install real-company-website
```

## 구성
- `PLAYBOOK.md` — 판별 신호 전체 목록·교정 기법·근거(외부 소스 인용)
- `plugins/real-company-website/skills/` — 스킬 6종(SKILL.md)
- `checklists/` — 카피·시각 신호 체크리스트(스킬 없이도 사용 가능)

## 핵심 원칙 (요약)
1. **특이성은 진짜 사실에서만 나온다** — 오너·실무자의 사실(수치·규격·원문)을 먼저 확보하고, 없는 사실은 지어내지 않는다.
2. **부제 생존 기준은 하나**: 새로운 사실을 주는가. 아니면 제목만 남긴다.
3. **색은 팔레트가 아니라 규칙**: 액션색 1종, 주색 2단계, 기능에만 유채색.
4. **사진은 실물만**: 스톡·AI 일러스트는 그 자체가 신호다.
5. **벤치마크는 감상이 아니라 문법 추출**: 실측 근거 없는 항목은 버린다.
6. **교정은 실측으로 끝난다**: 렌더에서 제목 재탕 0·자기 서술 0·어미 카운트를 수치로 확인.

## 라이선스
MIT
