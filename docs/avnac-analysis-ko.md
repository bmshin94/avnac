# Avnac 프로젝트 분석 정리 (한국어)

> 이 문서는 Avnac 저장소를 직접 읽고 분석한 내용을 한국어로 정리한 것입니다.
> 처음 이 프로젝트를 접하는 사람이 "이게 뭔지 / 어떻게 쓰는지 / 뭘 할 수 있는지"를
> 빠르게 파악할 수 있도록 작성했습니다.

**작성일:** 2026-09-17

---

## 📎 저장소 주소

| 구분 | 주소 |
|---|---|
| 이 저장소 (포크) | https://github.com/bmshin94/avnac |
| 원본 저장소 (upstream) | https://github.com/akinloluwami/avnac |

---

## 1. 이게 뭔가요?

**Avnac은 브라우저에서 돌아가는 디자인 에디터입니다.**
쉽게 말해 **캔바(Canva) / 미리캔버스의 오픈소스 버전**입니다.

포스터, SNS 카드뉴스, 유튜브 썸네일 같은 이미지를 설치 없이 웹브라우저에서 만들고
PNG / JPG / WebP로 내보낼 수 있습니다.

### 핵심 특징

- **로컬 우선(Local-first)** — 작업 파일이 서버가 아니라 **브라우저의 IndexedDB**에 저장됩니다.
  - 장점: 로그인 없이 바로 사용 가능, 서버 비용 거의 0원
  - 단점: 브라우저 저장소를 지우면 작업물도 사라집니다
- **데스크탑 전용** — 모바일 편집은 의도적으로 차단되어 있습니다
- **외부 캔버스 라이브러리 미사용** — Fabric.js 같은 라이브러리 없이 좌표 계산, 회전,
  스냅(자석 정렬), 베지어 곡선까지 직접 구현했습니다
- **라이선스: AGPL-3.0-only** (자세한 내용은 5번 항목 참고)

---

## 2. 폴더 구조

```text
avnac/
├── frontend/   ← 실제 에디터 화면 전부 (React 19 + Vite + TypeScript + Tailwind v4)
├── backend/    ← 보조 서버 (Elysia + Drizzle + PostgreSQL)
├── services/   ← AI 배경 제거 서버 (Python + FastAPI + RMBG-2.0)
├── docker/     ← rembg 도커 설정
└── vercel.json ← 배포 설정
```

### frontend — 이 프로젝트의 90%

| 경로 | 역할 |
|---|---|
| `src/lib/avnac-scene.ts` (1392줄) | 캔버스 위 도형/텍스트/이미지를 JSON으로 표현하는 **데이터 모델** |
| `src/lib/avnac-scene-render.ts` (789줄) | 화면에 그리고 이미지로 내보내는 **렌더링 엔진** |
| `src/scene-engine/primitives/` | 기하 계산, 스냅, 변형(transform) 로직 |
| `src/components/scene-editor/` | 툴바, 레이어 패널, 선택 오버레이 등 에디터 UI |
| `src/lib/avnac-ai-tambo-tools.ts` (605줄) | **AI 에이전트용 도구 정의 13개** (6번 항목 참고) |
| `src/data/artboard-presets.ts` | 캔버스 크기 프리셋 (현재 9개) |
| `src/data/google-font-families.ts` | 구글 폰트 목록 (1,526개) |
| `src/__tests__/` | 회귀 테스트 5종 |

**주요 라우트**

- `/` 랜딩 페이지
- `/files` 내 파일 관리
- `/create` 에디터 (실제 작업 화면)
- `/sponsor`, `/studio`, `/remove-bg`

### backend — 없어도 대부분 동작

| 파일 | 역할 |
|---|---|
| `src/routes/media.ts` (622줄) | 외부 이미지 CORS 프록시 (내보내기 깨짐 방지) |
| `src/routes/unsplash.ts` | 무료 사진 검색 |
| `src/routes/documents.ts` | 서버 측 문서 저장 |
| `src/routes/sponsor.ts` (394줄) | Paystack 결제 기반 후원 기능 |

### services — 현재 비활성화

Python + PyTorch로 `briaai/RMBG-2.0` 모델을 돌려 사진 배경을 제거하는 서버입니다.
**현재 기능 플래그로 꺼져 있습니다.** 코드에 남은 사유:

> "서버 비용이 무료 오픈소스 프로젝트가 감당하기에 너무 비싸서 배경 제거를 내렸습니다.
> Avnac이 유용하셨다면 후원을 고려해 주세요."

---

## 3. 설치 및 사용법

### 가장 쉬운 방법 (프론트엔드만)

```bash
cd frontend
npm install
npm run dev
```

→ 브라우저에서 `http://localhost:3300` 접속

**이것만으로 가능한 것:** 캔버스 생성, 텍스트/도형/이미지 편집, 정렬·회전·자르기·
그림자·블러, PNG/JPG/WebP 내보내기, 파일 관리(`/files`), JSON 가져오기/내보내기

### 백엔드까지 실행

```bash
cd backend
npm install
cp .env.example .env
npm run dev
```

→ `http://localhost:3001`

> ⚠️ **주의:** `.env`의 `DATABASE_URL`이 **필수**입니다.
> PostgreSQL이 없으면 서버가 아예 기동되지 않습니다.
> (`backend/src/config/env.ts`의 `z.string().min(1, 'DATABASE_URL is required')`)

### 기타 명령어

```bash
# 프론트엔드
npm run build       # 빌드
npm test            # 테스트 (vitest)
npm run lint        # biome 검사

# 저장소 루트 (양쪽 모두 설치한 경우)
npm run lint
npm run format:check
```

---

## 4. 이건 플러그인/스킬/MCP가 아닙니다

자주 하는 오해라 명확히 정리합니다.

| 종류 | 정체 | Avnac? |
|---|---|---|
| 플러그인 | 기존 프로그램에 끼우는 부품 | ❌ |
| 스킬 | AI에게 작업 방식을 가르치는 설명서 | ❌ |
| MCP | AI가 외부 도구를 쓰게 해주는 연결 규약 | ❌ |
| **독립 웹앱** | 그 자체로 완성된 프로그램 | ✅ **이것** |

Avnac은 **독립적으로 동작하는 웹 애플리케이션**입니다.
다만 프로젝트 **내부에** AI 기능이 포함되어 있습니다 (6번 항목).

> 참고: 이 포크의 루트에 있는 `CLAUDE.md`는 원본에 없던 파일로,
> 포크 소유자가 AI 어시스턴트 페르소나 설정을 위해 추가한 것입니다.

---

## 5. API 토큰이 필요한가요?

**기본 사용에는 토큰이 하나도 필요 없습니다.**

### 토큰 없이 되는 것

캔버스 편집, 텍스트/도형, 내 이미지 업로드, 저장, 이미지 내보내기 — 전부 무료이며
모두 브라우저 안에서 처리됩니다.

### 토큰이 필요한 부가 기능

| 기능 | 환경변수 | 비고 |
|---|---|---|
| Unsplash 사진 검색 | `UNSPLASH_ACCESS_KEY` | 무료 발급 |
| 후원 결제 | `PAYSTACK_SECRET_KEY` | 나이지리아 결제사 (국내 사용 불가) |
| 사용자 분석 | `VITE_PUBLIC_POSTHOG_PROJECT_TOKEN`, `VITE_PUBLIC_POSTHOG_HOST` | 선택 |
| AI 디자인 패널 | `VITE_TAMBO_API_KEY` | 현재 UI에서 숨김 상태 |
| 배경 제거 | `VITE_REMOVE_BG_ENABLED=true` + `BRIA_RMBG_URL`/`REMBG_URL` | 현재 비활성 |
| 프로 아이콘 | `HUGEICONS_NPM_TOKEN` | 없으면 무료 아이콘 자동 사용 |
| 로그인 / DB | `DATABASE_URL`, `BETTER_AUTH_SECRET` | 백엔드 실행 시 필수 |

> `HUGEICONS_NPM_TOKEN`은 **절대 `VITE_` 접두사를 붙이면 안 됩니다.**
> 설치/빌드 시점에만 필요하며, 붙이면 클라이언트 번들에 노출됩니다.

---

## 6. AI 에이전트 개발에 참고할 점

`frontend/src/lib/avnac-ai-tambo-tools.ts`에 **AI가 캔버스를 조작하는 도구 13개**가
이미 정의되어 있습니다.

```
describe_canvas    현재 캔버스 상태 조회
search_unsplash    사진 검색
add_unsplash_photo 검색한 사진 배치
add_rectangle      사각형 추가
add_ellipse        타원 추가
add_text           텍스트 추가
add_line           선 추가
add_image          이미지 추가
update_object      객체 수정
delete_object      객체 삭제
set_background     배경 설정
clear_canvas       전체 지우기
select_objects     객체 선택
```

### 배울 수 있는 패턴

**1) Zod 스키마 + `.describe()`로 도구 설명하기**

```ts
x: z.number().describe('X in artboard pixels.').optional(),
origin: z
  .enum(['top-left', 'center'])
  .describe("Whether x/y refers to the object's top-left corner (default) or geometric center.")
  .optional(),
```

**2) ref로 "살아있는 UI"에 안전하게 접근하기**

`avnac-ai-controller.ts` 주석에 설명된 설계:

> 모든 도구는 `MutableRefObject<AiDesignController | null>`로부터 지연 생성되어
> 패널이 리마운트되어도 살아남고 항상 현재 캔버스와 통신합니다.

**3) 입력값 방어적 검증**

```ts
.refine(isImageSourceString, { message: 'Must be http(s) URL or data:image/*;base64,...' })
```

현재는 Tambo라는 외부 서비스에 연결되어 있고 UI에서 숨겨져 있지만,
**다른 LLM API로 교체하면 그대로 동작하는 구조**입니다.

---

## 7. 기술 스택 정리

### 프론트엔드
React 19 · Vite 8 · TypeScript 5.7 · Tailwind CSS v4 · TanStack Router ·
Zustand · Zod 4 · Motion · PostHog · Hugeicons · jsPDF · qrcode · Vitest

### 백엔드
Elysia · Drizzle ORM · PostgreSQL · better-auth · Zod · TypeBox · tsx

### 서비스
Python · FastAPI · PyTorch · transformers (briaai/RMBG-2.0)

### 공통
Biome (lint + format) · Vercel (배포)

---

## 8. 라이선스 (AGPL-3.0) 주의사항

AGPL은 일반 GPL과 달리 **"웹서비스로 제공할 때도"** 소스 공개 의무가 발생합니다.

| 하는 일 | 소스 공개 의무 |
|---|---|
| 로컬에서 혼자 사용 | 없음 |
| 사내 내부 도구로 구축 | 사실상 없음 (이용자=직원에게 제공하면 충족) |
| 템플릿·폰트 등 에셋 판매 | 없음 (에셋은 코드가 아닌 별개 저작물) |
| 구축·커스터마이징 용역 | 없음 (파는 것이 노동력) |
| 수정한 코드로 공개 SaaS 운영 | **있음** |
| 코드를 숨기고 판매 | **라이선스 위반** |

> 핵심 원칙: **코드는 공개하고, 코드가 아닌 것(템플릿·용역·크레딧)으로 수익을 만든다.**

기여자 구성상 저작권이 사실상 한 명(Akinkunmi, 122커밋)에게 집중되어 있어,
필요하다면 원저작자와 별도의 상용 라이선스를 협의하는 선택지도 존재합니다.

---

## 9. 현재 상태와 개선 여지

### 확인된 사실

- 이 포크의 `HEAD~1`(`dc9cc8b`)이 원본 `main`의 최신 커밋과 **동일** → 포크는 최신 상태
- 원본의 마지막 개발 커밋은 **2026-05-11** → 이후 신규 커밋 없음
- 배경 제거 기능은 서버 비용 문제로 비활성화됨

### 비어 있는 부분 (= 개선 기회)

| 항목 | 현재 | 개선안 | 난이도 |
|---|---|---|---|
| **템플릿** | **0개** (템플릿 시스템 자체가 없음) | 템플릿 저장/불러오기 + 갤러리 | 중 |
| 한글 폰트 | 목록에는 있음(`Noto Sans KR`, `Jua`, `Black Han Sans` 등) | **1,526개 중에 섞여 있어 찾기 어려움** → 한글 폰트 그룹 분리 | 하 |
| 캔버스 프리셋 | 9개, 전부 해외 규격 | 네이버 블로그·스마트스토어·카카오채널 등 국내 규격 추가 | 하 |
| UI 언어 | 영어만 | 한국어 번역 | 하 |
| 결제 | Paystack (나이지리아) | 국내 PG(토스페이먼츠/포트원)로 교체 | 중 |
| AI 패널 | 숨김 + 외부 서비스 의존 | 다른 LLM API로 교체 후 활성화 | 중 |

### 비용 구조 (중요)

| 부분 | 서버 비용 |
|---|---|
| 캔버스 편집 / 저장 / 내보내기 | **0원** (전부 브라우저에서 처리) |
| 정적 호스팅 | 거의 0원 |
| 이미지 프록시 | 소액 (트래픽 비례) |
| **AI / 배경 제거** | **높음 (GPU 필요)** ← 원본이 중단한 이유 |

> 무료로 제공할 부분과 과금할 부분을 나누는 기준이 이미 명확합니다.
> 비용이 발생하지 않는 편집 기능은 무제한 무료로, GPU 비용이 드는 AI 기능만
> 크레딧/종량제로 운영하는 것이 안전합니다.

---

## 10. React / PHP로 만들 수 있나요?

### React

**이미 React 19로 만들어져 있습니다.** 새로 만들 필요 없이 이 코드를 수정해 쓰는 편이
훨씬 빠릅니다.

### PHP

절반만 가능합니다.

```text
┌──────────────────────────────┐
│ 브라우저 (에디터 화면)          │ ← PHP 불가. JS/React만 가능
│ 마우스 드래그, 실시간 렌더링     │
├──────────────────────────────┤
│ 서버 (저장·로그인·결제)         │ ← PHP 가능
└──────────────────────────────┘
```

PHP는 서버에서 실행되고 응답을 반환하는 언어라, 마우스 움직임마다 실시간으로 반응해야
하는 캔버스 편집기는 만들 수 없습니다. 다만 **백엔드를 PHP(Laravel 등)로 교체하는 것은
충분히 가능**합니다. 현재 백엔드 라우트가 `media` / `unsplash` / `documents` / `sponsor`
4개뿐이라 이식 부담이 크지 않습니다.

---

## 11. 수익화 아이디어 (검토 메모)

> ⚠️ 아래 금액과 시장 판단은 **검증된 시장조사 데이터가 아니라 추정치**입니다.
> 실행 전 반드시 직접 검증이 필요합니다.

AGPL 특성상 **"코드가 아닌 것"을 파는 모델**이 안전합니다.

| 방향 | 대상 | 수익 모델 | 난이도 | AGPL 리스크 |
|---|---|---|---|---|
| **쇼핑몰 상세페이지 자동 생성** | 스마트스토어·쿠팡 1인 셀러 | 구독 + 템플릿 판매 | 중 | 중 (SaaS면 코드 공개 필요) |
| **기업 내부 구축(SI)** | 프랜차이즈 본사, 대행사 | 구축비 + 유지보수비 | 하 | 낮음 |
| **한국형 툴 + 템플릿 마켓** | 일반 사용자 | 템플릿 팩·구독·수수료 | 중 | 낮음 |
| **AI 디자이너** | 전체 | 크레딧 종량제 | 하 (도구 13개 기구현) | 낮음 |
| **유튜브 썸네일 특화** | 크리에이터 | 템플릿 구독 | 하 | 낮음 |

### 기술적으로 이미 유리한 지점

- 장면(scene)이 **JSON 구조**라, 템플릿의 `{{변수}} 치환 → 렌더 → 내보내기` 파이프라인을
  얹기 쉽습니다. 상세페이지·썸네일 자동 생성의 핵심 기반이 이미 존재합니다.
- 편집·저장·내보내기가 전부 브라우저에서 처리되어 **서버 비용이 거의 들지 않습니다.**
- AI 도구 13개가 이미 정의되어 있어 LLM 교체만으로 AI 기능을 되살릴 수 있습니다.

### 반드시 지켜야 할 3가지

1. 코드는 공개하고, **템플릿·용역·크레딧**으로 수익을 만든다.
2. **AI 기능은 반드시 종량제(크레딧)로 한다.** 원본이 무제한 무료로 운영하다
   서버 비용을 감당하지 못해 기능을 중단했다.
3. 순서는 **한국화 → 템플릿 시스템 → 결제 연동 → AI** 순으로 진행한다.
   앞의 두 단계는 비용이 들지 않으면서 결과가 즉시 보인다.

---

## 12. 참고 링크

- 이 저장소: https://github.com/bmshin94/avnac
- 원본 저장소: https://github.com/akinloluwami/avnac
- 기여 가이드: [`CONTRIBUTING.md`](../CONTRIBUTING.md)
- 프로젝트 개요(영문): [`README.md`](../README.md)
- 라이선스 전문: [`LICENSE`](../LICENSE)
