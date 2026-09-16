# mizchi/skills 분석 정리 (한국어)

> Agent Skills 레포지토리 분석 · 활용 가이드 · 수익화 전략
>
> 작성일: 2026-09-16

## 📍 저장소 주소

| 구분 | URL |
| --- | --- |
| **이 저장소 (포크)** | https://github.com/bmshin94/mizchi-skills |
| **원본 (upstream)** | https://github.com/mizchi/skills |
| 작업 브랜치 | `claude/affectionate-cerf-ss8mrk` |

관련 링크

- Agent Skills 표준: https://agentskills.io
- APM (Agent Package Manager): https://github.com/apm-sh/apm
- `skills` CLI: https://github.com/vercel-labs/skills
- waxa (스킬 평가 CLI): https://www.npmjs.com/package/@mizchi/waxa

---

## 1. 이게 뭔가요?

**한 줄 요약: AI 코딩 에이전트에게 읽히는 "업무 매뉴얼" 68개 묶음.**

AI는 똑똑하지만 우리 회사 사정은 모르는 신입사원과 비슷하다. 그냥 "리뷰해줘"
라고 하면 두루뭉술하게 답하지만, **절차가 적힌 매뉴얼**을 쥐여주면 갑자기
10년차처럼 일한다. 이 저장소가 바로 그 매뉴얼 뭉치다.

- 폴더 하나 = 스킬 하나 = 매뉴얼 한 권
- 각 폴더에 `SKILL.md`(설명서) + 선택적으로 `scripts/`, `assets/`, `references/`
- 일본 프론트엔드 개발자 **mizchi**가 실무 노하우로 작성
- 라이선스: **MIT 기본**, 일부 스킬은 자체 `LICENSE.txt`(Apache 2.0)

### `SKILL.md` 구조

```yaml
---
name: frontend-review-triage
description: Use when starting a frontend review engagement or when the user
  asks for an initial assessment ("triage", "day 0", ...). Reads package.json,
  README, gh issues, and produces a scorecard...
---

# Frontend Review — Triage
## Procedure   (0 → 1 → 2 → 3 단계별 절차)
## Output      (산출물 경로와 형식까지 지정)
## Boundaries  (하면 안 되는 일 — 이게 핵심!)
```

`description`이 **"언제 이 스킬을 꺼내 쓸지"** 알려주는 트리거다. 사용자가
직접 부르지 않아도 AI가 상황을 보고 알아서 골라 읽는다.

---

## 2. 스킬 목록 (총 68개)

| 카테고리 | 개수 | 대표 스킬 |
| --- | --- | --- |
| 🎨 `frontend/` | 14 | review-triage, review-security, review-weekly |
| 🛠 `tooling/` | 11 | justfile, nix-setup, dotenvx, ast-grep-practice |
| 🧠 `meta/` | 9 | skill-selector, waxa-eval, empirical-prompt-tuning |
| ☁️ `aws/` `cloudflare/` `k8s/` `devops/` | 14 | ecs-service-connect-ipv6, gh-fix-ci, opentelemetry |
| 🗄 `sql/` | 5 | lint, plan-audit, schema-audit, security |
| 💻 `lang/` | 5 | moonbit-practice, gleam-practice, ts2moonbit-migration |
| 🧪 `testing/` `ai/` `formal-methods/` | 6 | playwright-test, review-image, vlmkit |
| 🔧 `tools/` | 1 | waxa (스킬 평가 CLI, npm 배포됨) |

### 알짜: `frontend/review-*` 세트

실제 프론트엔드 컨설팅 업무 프로세스를 그대로 스킬화했다.

| 스킬 | 역할 |
| --- | --- |
| `review-triage` | Day 0 진단 — 점수표 + 위험요소 Top 3 |
| `review-ci` | CI 병목 분석 — 샤딩/캐시/동시성 처방 |
| `review-hygiene` | TypeScript 엄격도, 린트, 데드코드, 중복 |
| `review-deps` | 의존성 건강도 — CVE를 **공격 벡터 가중치**로 분류 |
| `review-testing` | vitest 커버리지, Playwright 설정, VRT |
| `review-security` | HTML 싱크, 토큰 저장, 라우트 가드, env 노출 |
| `review-state` | 상태관리 아키텍처 — Jotai/Zustand/Redux 안티패턴 |
| `review-performance` | 렌더링 성능 — 프로파일러 우선, memo 정확성 |
| `review-weekly` | **오케스트레이터** — 위 전부 실행 + KPI 비교 + 이슈 등록 |

`review-triage`는 앱 유형(`admin`/`toc`/`btob-saas`/`ec`/`fintech`/
`healthcare`/`iot-ops`/`media`)별로 어떤 검사가 P0/P1/생략인지 판단하는
**우선순위 매트릭스**를 갖고 있다. 규제 맥락(GDPR, PCI DSS, HIPAA)도 반영한다.

### `review-weekly`의 5단계 파이프라인

```
Phase 1    6개 스킬 순차 실행 → 원시 JSON 수집
Phase 1.5  아키텍처 리뷰 (state, performance)
Phase 2    5개 전문가 관점 에이전트 병렬 실행
           (frontend / react / performance / security / ops expert)
Phase 3    지난 주기 KPI와 비교 → 회귀(regression) 감지 + 래칫(ratchet)
Phase 4    3주 연속 발견된 문제 → 정적 규칙(eslint/codemod/CI 게이트) 승격 제안
Phase 5    리포트 작성 + `gh issue create`로 이슈 자동 등록
```

**래칫(ratchet) 개념**: 지표가 좋아지면 기준선을 그만큼 올려 잠근다. 다시
나빠질 수 없게 만드는 구조 — "한 번 진단"이 아니라 "계속 좋아지는 시스템".

---

## 3. 설치 및 사용법

### 설치 (택 1)

```bash
# 방법 A — APM (공식 추천). 전역 설치 시 ~/.claude/skills/ 로 들어간다
apm install -g mizchi/skills/frontend/review-triage

# 방법 B — skills CLI (감지된 모든 에이전트에 한 번에 설치)
npx skills add mizchi/skills --skill frontend-review-triage

# 방법 C — 그냥 폴더 복사
npx degit mizchi/skills/tooling/justfile ~/.claude/skills/justfile
```

프로젝트 단위로 관리하려면 `apm.yml`:

```yaml
name: my-project
targets:
  - claude
dependencies:
  apm:
    - mizchi/skills/frontend/review-triage
    - mizchi/skills/devops/gh-fix-ci#v0.1.0   # 태그 고정
  mcp:
    - io.github.github/github-mcp-server       # MCP는 별도 칸
```

### 사용법 → 별도로 없음

설치 후에는 아무것도 안 해도 된다. AI가 각 스킬의 `description`을 보고
상황에 맞는 것을 자동으로 읽는다. 물론 이름으로 직접 호출해도 된다.

### ⚠️ 이 저장소를 수정할 때 주의 (`CLAUDE.md` 규칙)

- 여기서 파일을 고치면 `~/.claude/skills/<이름>/` 에도 **수동으로 미러링**해야
  현재 세션이 변경을 인식한다 (자동 반영 안 됨)
- 커밋 전 `pkf hooks install` 한 번 → frontmatter 검증 훅 설치
- 폴더 이름 == `SKILL.md`의 `name:` 값이어야 한다 (CI가 검사)
- `pkf run test` 로 검증 테스트 + frontmatter 체크 실행
- `SKILL.md`는 영어로 작성 (공개 저장소 규칙). 일본어 원본은 `SKILL-ja.md` 유지

---

## 4. 스킬 vs MCP vs 플러그인

**이 저장소는 "스킬"이다.** 셋은 완전히 다른 개념이다.

| | **Skill** (이 저장소) | **MCP** | **Plugin** |
| --- | --- | --- | --- |
| 정체 | 마크다운 문서 폴더 | 실행 중인 서버 프로세스 | 스킬+MCP+명령어 묶음 |
| 주는 것 | **지식/절차** | **능력** (새 도구·데이터 접근) | 여러 개 세트 |
| 비유 | 📖 레시피북 | 🔌 새 조리도구 설치 | 📦 세트 상품 |
| 실행 | 안 함, 읽히기만 함 | 프로세스가 돈다 | 조합에 따라 다름 |
| 토큰 필요 | ❌ | 보통 ⭕ | 경우에 따라 |

**근거**: `apm.yml` 스키마가 `dependencies.apm`(스킬)과 `dependencies.mcp`
(MCP 서버)를 명확히 분리해서 정의한다.

스킬은 실행 코드가 아니라 문서이므로 가볍고, 어느 에이전트에나 붙고,
보안 위험도 상대적으로 낮다.

---

## 5. API 토큰이 필요한가?

**68개 중 3개만 필요하다. 나머지 65개는 키가 0개.**

| 스킬 | 필요한 키 | 이유 |
| --- | --- | --- |
| `ai/review-image` | `OPENROUTER_API_KEY` | 스크린샷을 비전 모델에 전송 |
| `ai/vlmkit` | `OPENROUTER` / `GEMINI` / `ANTHROPIC` 중 1개 | VLM 호출 |
| `node/pi-coding-agent` | `ANTHROPIC_API_KEY` | 가이드 문서일 뿐, 스킬 자체가 호출하진 않음 |

### 헷갈리기 쉬운 것

- `cloudflare/`, `aws/` 폴더의 `CF_TOKEN`, `API_KEY` 등은 **"토큰을 이렇게
  다루세요"라는 예제 코드**다. 스킬이 요구하는 게 아니라 가르쳐주는 내용.
- `tools/waxa`는 `claude` CLI가 인증돼 있으면 된다. **OAuth 로그인으로 충분**
  하며 별도 API 키 구매가 필수는 아니다.

**결론: Claude Code 구독만 있으면 추가 비용 없이 거의 전부 사용 가능.**

---

## 6. 왜 GitHub에서 주목받나

1. **제작자 인지도** — mizchi는 일본 프론트엔드 씬의 네임드. "이 사람이 실제로
   쓰는 세팅"이라는 것 자체가 콘텐츠 가치를 갖는다.
2. **타이밍** — Agent Skills가 뜨면서 다들 "뭘 만들지" 고민하던 시점에 등장.
3. **실무 깊이** — 대부분의 스킬 저장소가 Hello World 수준일 때, 이건 앱 유형별
   우선순위 매트릭스, 단계별 절차, **Boundaries(금지선)** 까지 갖췄다.
4. **평가 도구를 직접 만듦** — `tools/waxa`로 스킬 성능을 정량 측정(스킬 있을
   때 vs 없을 때 A/B)하고 npm에 배포까지 했다. "스킬 품질을 어떻게 검증하나"
   라는 공통 고민에 실행 가능한 답을 준 것이 결정타.
5. **희소 지식** — 구글링해도 안 나오는 함정들이 축적돼 있다.
   - ECS Service Connect가 IPv4 전용 Fargate에 IPv6 주소를 반환하는 문제
   - Node 24 `node:sqlite` + sqlite-vec 사용 시 vitest가 조용히 깨지는 이유
   - esbuild ESM 환경에서 OTel이 무음 실패하는 함정
6. **락인 없음** — agentskills.io 표준이라 Claude Code, Codex, Cursor,
   Gemini CLI, opencode 모두 호환.

> 참고: 원본 저장소의 스타 수는 이 세션의 접근 범위 밖이라 직접 확인하지
> 못했다. 위는 구조적 요인 분석이다.

---

## 7. 로컬 에이전트 구축에 도움이 되나 → 된다

### 레벨 1: 기존 에이전트 강화 (난이도 ⭐)

`~/.claude/skills/`에 복사하면 끝. 오늘 당장 가능.

### 레벨 2: 스킬 시스템 설계 배우기 (난이도 ⭐⭐) — 진짜 가치

`meta/` 폴더가 사실상 **에이전트 설계 교과서**다.

| 스킬 | 배울 수 있는 것 |
| --- | --- |
| `empirical-prompt-tuning` | 프롬프트를 감이 아니라 실험으로 개선 |
| `optimizing-descriptions` | **어떻게 써야 AI가 제때 스킬을 꺼내는가** (최난제) |
| `waxa-eval` | 평가 루프 설계 (시나리오 → 채점 → 개선 반복) |
| `skill-selector` / `skill-finder` | 스킬 검색·선택 알고리즘 (2단계 폴백) |
| `retrospective-codify` | 삽질 경험 → 자동화 규칙으로 변환 |

로컬 에이전트를 만들면 반드시 부딪히는 문제가 "지식을 어떻게 넣고, 언제
꺼내게 할까"인데, 이 폴더가 그 답을 갖고 있다.

### 레벨 3: 에이전트 런타임 직접 구축 (난이도 ⭐⭐⭐)

`node/pi-coding-agent` 스킬이 정확히 이 주제를 다룬다.

- `@mariozechner/pi-coding-agent`를 Node에 임베드
- `pi.registerTool` / `pi.registerCommand` / `pi.on` 으로 커스텀 툴 등록
- SDK 모드 vs `pi --mode rpc` 선택 기준
- 실제 함정: `~/.pi/agent/auth.json`이 환경변수를 가려버리는 문제,
  cwd × tools 팩토리 트랩, TypeBox 툴 스키마, `peerDependencies` 규칙

### 보너스: 아키텍처 패턴

`review-weekly`의 오케스트레이터 + 병렬 서브에이전트 구조는 멀티 에이전트
설계 시 그대로 참고할 수 있다.

---

## 8. React / PHP로 만들 수 있나

### 해석 A: React/PHP용 스킬을 만들 수 있나 → **가능, 5분이면 된다**

스킬은 마크다운 폴더라 언어와 무관하다. 예시:

```bash
mkdir -p ~/.claude/skills/laravel-practice
```

`~/.claude/skills/laravel-practice/SKILL.md`:

```markdown
---
name: laravel-practice
description: Use when writing or reviewing Laravel/PHP code — Eloquent N+1
  queries, form request validation, service container bindings, Pest tests.
  Trigger on "라라벨", "Eloquent", "artisan", or .php files in app/.
---

# Laravel 코딩 규칙

## 절차
1. `composer.json`으로 Laravel 버전 확인
2. `php artisan about`으로 환경 파악
3. 아래 체크리스트로 검토

## 체크리스트
- [ ] Eloquent N+1 → `with()` eager loading 사용했나
- [ ] `$request->all()` 대신 FormRequest 사용하나
- [ ] 마이그레이션에 `down()` 있나
- [ ] `DB::raw()`에 문자열 직접 결합 없나 (SQL 인젝션)

## 금지
- `.env` 파일 커밋/수정 금지
- 프로덕션 마이그레이션 자동 실행 금지
```

`name:`은 폴더 이름과 같아야 하고, `description:`에 **언제 쓰는지**를 구체적
으로 써야 자동 발동이 잘 된다 (`meta/optimizing-descriptions` 참고).

### 해석 B: 이런 플랫폼을 React/PHP로 만들 수 있나 → 역할 분담 필요

| 부분 | React | PHP | 평가 |
| --- | --- | --- | --- |
| 스킬 관리 UI | ⭐⭐⭐ | ⭐⭐⭐ | 적합 |
| 마켓플레이스 웹 | ⭐⭐⭐ | ⭐⭐⭐ (Laravel) | 적합 |
| 리포트 대시보드 | ⭐⭐⭐ | ⭐⭐ | 적합 |
| CLI 도구 | ⭐ | ⭐ | Node/Deno/Go 권장 |
| **에이전트 런타임** | ❌ | ⚠️ | Node/Python 권장 |

PHP는 요청-응답 모델이라 몇 분씩 도는 **장시간 스트리밍 루프**에 약하다.
에이전트 SDK·MCP 라이브러리 생태계도 Node/Python이 압도적이다.

**권장 구조:**

```
┌─────────────────────────────────────┐
│  React (Next.js) — 웹 UI            │
│  스킬 편집기 / 리포트 대시보드      │
└────────────┬────────────────────────┘
             │ HTTP / SSE
┌────────────▼────────────────────────┐
│  Laravel — API + 인증 + 결제        │
│  사용자 / 구독 / 스킬 메타데이터    │
└────────────┬────────────────────────┘
             │ 큐 (Redis / SQS)
┌────────────▼────────────────────────┐
│  Node 워커 — 에이전트 실행          │
│  Claude Agent SDK + 스킬 로딩       │
└─────────────────────────────────────┘
```

React/PHP가 90%를 담당하고, 에이전트가 도는 부분만 Node 워커로 분리한다.

---

## 9. 수익화 전략

### 9.1 냉정한 전제

**스킬 자체는 거의 팔리지 않는다.**

| 문제 | 설명 |
| --- | --- |
| 복사가 쉬움 | 마크다운 텍스트. 한 명이 사서 뿌리면 끝 |
| 무료 대안 다수 | 이 저장소도 MIT 무료. Anthropic 공식 스킬도 무료 |
| AI가 대신 만듦 | "라라벨 스킬 만들어줘" 하면 5분 만에 생성됨 |
| 가격 앵커가 낮음 | "문서 쪼가리에 왜 돈을?" 인식 |

**핵심 통찰: 스킬은 상품이 아니라 원가 절감 장치다.**
남들이 40시간 걸리는 일을 8시간에 끝내는 것 — 그 차익이 마진이다.
스킬을 파는 게 아니라, 스킬로 무장한 사람을 파는 것.

### 9.2 가치 사슬 — 돈이 고이는 곳

```
[ 스킬 문서 ]  →  [ 실행 ]  →  [ 해석 ]  →  [ 책임 ]
   무료           저렴         비쌈        제일 비쌈
   복사 가능      토큰값       전문가 판단  리테이너/구독
```

오른쪽 두 칸(해석, 책임)을 노려야 한다.

### 9.3 아이디어 6개

#### ① AI 프론트엔드 건강검진 (일회성) — 등급 A+

고객 레포에 `frontend/review-*` 세트를 돌리고 **사람이 읽을 수 있는 리포트**로
가공해 판매.

- **타겟**: 시리즈 A 전 스타트업, 외주 인수 기업, 개발팀 없는 대표,
  M&A 기술 실사 (단가 최고)
- **가격(추정)**: Lite 30~50만 / **Standard 100~200만** / Deep 250~400만 /
  M&A 실사 500만~
- **원가**: 토큰 약 1~5만원 + 시간 8~12시간 → **시간당 12~18만원**
  (일반 프론트 외주 시간당 5~8만원의 2배 이상)
- **리스크**: "내 코드를 왜 보여주나" 저항(NDA + 읽기 전용 접근으로 대응),
  할루시네이션(전량 검수 필수)

#### ② 월간 코드 품질 구독 (리테이너) — 등급 S ⭐ 주력 추천

①을 매달 반복. `review-weekly`가 이미 이 목적으로 설계돼 있다.

| | 일회성 ① | 구독 ② |
| --- | --- | --- |
| 매출 | 들쭉날쭉 | 예측 가능 |
| 영업 | 매번 새 고객 | 한 번 계약 = 12개월 |
| 원가 | 매번 세팅 | 2회차부터 절반 이하 |
| 가치 | "문제를 알려줌" | "**좋아지는 것을 증명함**" |

- **가격(추정)**: Basic 30~50만 / **Pro 80~120만** / Partner 200~300만 (월)
- 고객 5곳 × Pro 100만 = 월 500만, 2회차부터 회당 3~4시간
- **결정적 강점**: 3개월 후 이런 그래프를 제시할 수 있다

```
TypeScript strict 위반:  142 → 89 → 31
CI 평균 시간:            14분 → 11분 → 7분
고위험 CVE:              8 → 3 → 0
```

지불한 돈이 한 일이 숫자로 증명되므로 해지율이 낮다.

#### ③ 한국 특화 스킬팩 — 등급 B+ (미끼로는 A)

이 저장소에 한국 개발 환경 스킬이 **0개**다. 빈 시장.

```
korea-agent-skills/
├─ payments/toss-payments/      결제위젯, 웹훅 검증, 부분취소
├─ payments/portone-v2/         포트원 V2 마이그레이션 함정
├─ auth/kakao-login/            카카오싱크, 동의항목
├─ auth/naver-login/
├─ compliance/pipa-review/      개인정보보호법 점검
├─ compliance/e-commerce-law/   전자상거래법 필수 표기
├─ infra/naver-cloud/
└─ ecommerce/coupang-api/
```

- **구조**: 기본팩 무료 공개 → 인지도 → ①② 고객 유입 / 기업 전용 프라이빗
  팩은 월 20~50만
- **왜 결제·법규인가**: 틀리면 돈이 날아가거나 과태료를 맞는 영역이라
  지불 의사가 명확하다. "코드 예쁘게"는 팔리지 않는다.
- **주의**: 유지보수 부담 때문에 처음엔 **5개 이내로만**

#### ④ 사내 노하우 → 스킬 전환 대행 (B2B) — 등급 A (2단계)

고객의 고통: "시니어가 아는 걸 주니어가 모른다. 위키에 써놨는데 안 읽는다."
→ AI가 읽는 문서로 바꾸면 주니어가 안 읽어도 AI가 읽고 알려준다.

방법론이 이미 저장소에 있다: `retrospective-codify`,
`optimizing-descriptions`, `empirical-prompt-tuning`, `waxa`.

**waxa가 킬러**: 컨설팅의 영원한 약점인 "효과 측정 불가"를 A/B 델타로 해결.

- **가격(추정)**: 워크숍 200~300만 / **구축 800~1500만** / 운영 월 100~200만
- **리스크**: 의사결정자가 CTO급이라 영업 난이도 높음 → ①②로 신뢰를 쌓은 뒤
  업셀하는 것이 정석

#### ⑤ 스킬 마켓플레이스 / 품질 레지스트리 — 등급 C+

`meta/skill-finder`를 보면 이미 6개 소스가 경쟁 중이다 (Anthropic 공식,
claude-skill-registry, VoltAgent, ComposioHQ, Superpowers, GitHub 토픽).

- **차별점**: 어느 마켓도 "실제 효과"를 측정하지 않는다. waxa를 자동 실행해
  점수를 매기는 **"스킬계의 Lighthouse"** 포지션이 비어 있다.

```
기존:  ⭐ 1.2k stars, 5k downloads
차별:  ⭐ 1.2k | 효과 델타 +41% | 오발동률 3% | 토큰 효율 A
```

- **리스크(높음)**: 닭-달걀 문제, 평가 비용이 토큰값으로 실시간 발생,
  6개월~1년 선투자 필요
- React 실력을 살릴 수 있는 유일한 아이템이긴 하다

#### ⑥ 교육 콘텐츠 + 퍼스널 브랜딩 — 등급 B (연료)

"Agent Skills 만드는 법" 한국어 자료가 거의 없다. 선점 타이밍.

| 형태 | 가격 |
| --- | --- |
| 블로그/유튜브 | 무료 (미끼) |
| 전자책 (Gumroad) | 2~4만원 |
| 인프런/클래스101 | 8~15만원 |
| 기업 출강 | 반나절 150~300만 |

**진짜 목적은 판매가 아니라 영업 자산**이다. ①②④ 고객이 여기서 나온다.
mizchi가 하는 것이 정확히 이 경로 — 공개 → 인지도 → 컨설팅/강연/제안.

### 9.4 비교표

| # | 아이디어 | 시작속도 | 수익성 | 지속성 | 리스크 | 종합 |
| --- | --- | :---: | :---: | :---: | :---: | :---: |
| ① | 일회성 건강검진 | 높음 | 높음 | 중 | 낮음 | **A+** |
| ② | **월간 구독** | 중 | 높음 | 매우높음 | 낮음 | **S** |
| ③ | 한국 스킬팩 | 높음 | 중 | 중 | 낮음 | **B+** |
| ④ | 노하우 전환 대행 | 낮음 | 매우높음 | 높음 | 중 | **A** |
| ⑤ | 마켓플레이스 | 낮음 | 높음(성공시) | 높음 | 높음 | **C+** |
| ⑥ | 교육/브랜딩 | 높음 | 중 | 높음 | 낮음 | **B** |

### 9.5 추천 조합 — 3층 구조

```
3층 │  ④ 사내 노하우 전환 (800~1500만)   ← 신뢰 쌓인 뒤 업셀
2층 │  ② 월간 구독 (월 80~120만)         ← 주력 매출
1층 │  ① 일회성 진단 (100~200만)         ← 진입점
0층 │  ③ 한국 스킬팩 + ⑥ 콘텐츠 (무료)   ← 미끼
```

흐름: 무료로 신뢰 → 진단 1회 → "매달 봐드릴까요?" → 구독 전환 → 대형 컨설팅

### 9.6 90일 실행 로드맵

**Day 1–14: 무기 만들기**
- `frontend/review-*` 전체 설치
- 내 프로젝트 + 지인 프로젝트 3개에 실행
- **실측**: 소요 시간, 토큰 비용, 할루시네이션 비율 (가격 산정 근거)
- 리포트를 "개발 모르는 대표도 읽을 수 있게" 재가공 ← 진짜 부가가치

**Day 15–30: 증거 만들기**
- 샘플 리포트 1개 익명화 공개 (블로그)
- 한국 특화 스킬 3개만 제작해 GitHub 공개 (토스 / 카카오 / PIPA)
- 링크드인·커뮤니티에 후기 글

**Day 31–60: 첫 고객** ← 전체 계획에서 가장 중요
- 지인 회사 1곳 **무료** 진단 → 후기 + 케이스 스터디 확보
- 두 번째 고객은 반값
- 랜딩페이지(React) + NDA 템플릿 + 계약서 준비

**Day 61–90: 구독 전환**
- 진단 고객 전원에게 월간 추적 제안
- `review-weekly`로 2회차 자동화 → 소요 시간 반감 확인
- 목표: 구독 고객 2~3곳 (월 200~300만 안정 매출)

### 9.7 법적 / 품질 체크리스트

| 항목 | 해야 할 일 |
| --- | --- |
| **라이선스** | MIT / Apache 2.0 → 상업 이용 허용. **저작권 표시 유지 필수**. 리포트 부록에 "based on mizchi/skills (MIT)" 명시 |
| **NDA** | 고객 코드 열람 전 필수. 표준 템플릿 준비 |
| **접근 권한** | **읽기 전용만** 요구. 쓰기 권한은 받지 않는다 |
| **데이터 처리** | 코드가 외부 LLM 서버로 전송됨 → **고객에게 명시적 고지 및 동의** 필수 |
| **면책 조항** | "AI 보조 분석이며 완전성을 보증하지 않음" 계약서에 명시 |

품질 원칙

- AI 출력을 그대로 전달하지 않는다 — 전량 검수
- 확신 없는 항목은 "추가 확인 필요"로 표기
- 근거(파일:줄번호)를 반드시 첨부 — 검증 가능해야 신뢰가 생긴다
- 커버하지 못한 영역은 솔직히 밝힌다

### 9.8 손절 기준

| 신호 | 판단 |
| --- | --- |
| 무료 진단 3곳 후 후기·소개 없음 | 가치 전달 실패 → 리포트 재설계 |
| 진단 고객 5곳 중 구독 전환 0 | 구독 명분 부족 → ①만 하거나 피벗 |
| 회당 소요가 계속 20시간 초과 | 자동화 실패 → 단가 인상 또는 범위 축소 |
| 6개월간 유료 고객 0 | 시장 미성숙 → ⑥(콘텐츠)로 전환 |

---

## 10. 전체 요약

| 질문 | 답 |
| --- | --- |
| 이게 뭔가 | AI가 읽는 업무 매뉴얼 68개 (Agent Skills) |
| 설치·사용법 | `apm install -g` 또는 폴더 복사. **사용법은 없음 — 자동 발동** |
| 스킬/MCP/플러그인 | **스킬**. 실행되지 않는 문서 폴더 |
| API 토큰 | 68개 중 3개만 필요. 나머지는 불필요 |
| 왜 유명한가 | 제작자 인지도 + 타이밍 + 실무 깊이 + 평가 도구(waxa) 자체 제작 |
| 로컬 에이전트 | `meta/`가 설계 교과서, `node/pi-coding-agent`가 런타임 구축 가이드 |
| React/PHP | 스킬 작성은 언어 무관. 플랫폼은 React + Laravel + Node 워커 조합 |
| 수익화 | **① 일회성 진단 → ② 월간 구독**을 주축으로, ③⑥을 미끼로, ④로 업셀 |

---

## 부록: 저장소 관리 정보

```bash
# 저장소 검증
pkf run test            # frontmatter 검증 테스트 + 체크 + README 동기화 확인
pkf run gen:readme      # 각 스킬 README 재생성
pkf run check:frontmatter
pkf hooks install       # pre-commit 훅 설치 (체크아웃당 1회)

# waxa (스킬 평가)
npx @mizchi/waxa init --skill <name>
npx @mizchi/waxa <skill>/evals/eval.yaml --baseline   # 스킬 유/무 A/B 비교
npx @mizchi/waxa iterate <skill>/evals/eval.yaml
```

- 빌드 산출물(`_build/`, `.mooncakes/`, `node_modules/`)은 커밋 금지
- 보안 민감 정보가 생기면 이 공개 저장소에서 제거하고 chezmoi 로컬로 이동
- `npm-release/`, `moonbit-*`, `flaker-setup`, `ast-grep`, `tuimbt-practice`는
  이 저장소 소유가 아니다 (각자의 upstream에서 관리)
