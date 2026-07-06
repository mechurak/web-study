# 02. 렌더링 — SSR / SSG / ISR / CSR, 서버 컴포넌트

> **핵심 질문**
> - "어디서(서버/클라이언트), 언제(빌드 시/요청 시) HTML을 만드는가"로 4가지 방식을 구분할 수 있는가?
> - 서버 컴포넌트(RSC)와 SSR은 다른 개념이다 — 무엇이 다른가?
> - `"use client"`를 붙이는 기준은? 붙이면 무엇을 잃는가?

## 개념 정리

### 렌더링 방식 4가지 — "어디서 + 언제"로 구분

| 방식 | HTML 생성 시점 | 적합한 경우 | 특징 |
|---|---|---|---|
| **SSG** (Static) | 빌드 시 1회 | 블로그 글, 문서, 랜딩 페이지 | CDN에서 서빙 — 가장 빠르고 가장 싸다 |
| **ISR** (Incremental) | 빌드 시 + 주기적 재생성 | 상품 페이지, 뉴스 목록 | SSG 속도 + 데이터가 "적당히" 최신 |
| **SSR** (Dynamic) | 요청 시마다 | 내 대시보드, 검색 결과 | 항상 최신, 유저별 다른 내용 가능. 매 요청 서버 실행 |
| **CSR** | 브라우저에서 JS로 | 실시간 갱신 위젯, 상호작용 UI | 서버 부담 없음, 첫 페인트 늦음, SEO 약함 |

판단 기준은 두 가지 질문이다: **"모든 유저에게 같은 내용인가?"** (yes → SSG/ISR 후보), **"얼마나 최신이어야 하는가?"** (초 단위 → SSR/CSR, 분 단위 허용 → ISR).

App Router에서는 페이지 단위 함수(`getStaticProps` 등)가 아니라 **코드가 무엇을 쓰는지로 자동 결정**된다: 기본은 정적(SSG)이고, `cookies()`, `headers()`, `searchParams` 같은 요청 의존 API를 쓰거나 캐시를 끄면 그 라우트는 동적(SSR)이 된다. ISR은 `export const revalidate = 60`(초) 선언으로. 빌드 로그(`npm run build`)에 각 라우트가 `○`(static)인지 `ƒ`(dynamic)인지 표시되니 항상 확인하는 습관을 들일 것.

동적 라우트(`[id]`)를 SSG로 만들려면 어떤 id들이 존재하는지 빌드 타임에 알려줘야 한다 — `generateStaticParams()`.

### 서버 컴포넌트 vs 클라이언트 컴포넌트 — SSR과 다른 축이다

혼동하기 쉬운데, **SSR은 "첫 HTML을 서버에서 만든다"는 시점 이야기**고, **서버/클라이언트 컴포넌트는 "이 컴포넌트의 코드가 브라우저에 전송되느냐"는 코드 위치 이야기**다. 클라이언트 컴포넌트도 SSR 시 서버에서 첫 HTML은 그려진다 — 다만 그 코드가 번들에 포함돼 브라우저에서 다시 실행(hydration)된다는 점이 다르다.

| | 서버 컴포넌트 (기본값) | 클라이언트 컴포넌트 (`"use client"`) |
|---|---|---|
| 실행 위치 | 서버에서만 | 서버(첫 HTML) + 브라우저 |
| JS 번들 포함 | ❌ 안 감 | ✅ 감 |
| 할 수 있는 것 | `await` 데이터 조회, 시크릿 사용, fs 접근 | `useState`, `useEffect`, 이벤트 핸들러, 브라우저 API |
| 할 수 없는 것 | 상태, 이벤트, 브라우저 API | 서버 자원 직접 접근 |

`"use client"`를 붙이는 기준: **상호작용(클릭, 입력, 상태)이나 브라우저 API가 필요한 순간에만**. 붙이면 그 파일과 그것이 import하는 모든 것이 번들로 가고, 서버 자원 직접 접근을 잃는다.

### 경계 설계 — `"use client"`를 트리의 어디에 두는가

`"use client"`는 그 지점부터 하위 전체를 클라이언트로 만드는 **경계 선언**이다. 그래서 원칙은 **경계를 최대한 잎(leaf) 쪽으로 밀기**:

```tsx
// ❌ 페이지 전체를 클라이언트로 — 데이터 조회도 못 하게 됨
"use client"
export default function MemoPage() { ... }

// ✅ 페이지는 서버 컴포넌트, 상호작용 부분만 분리
export default async function MemoPage() {
  const memos = await getMemos()            // 서버에서 조회
  return <MemoList memos={memos} />          // 데이터는 props로
}
// MemoList.tsx 안의 좋아요 버튼만 "use client"
```

단, 반대 방향은 된다: 클라이언트 컴포넌트의 `children`으로 서버 컴포넌트를 **끼워 넣는** 것은 가능(레이아웃에서 자주 쓰는 패턴).

서버 → 클라이언트로 props를 넘길 때는 **직렬화 가능해야 한다**(JSON으로 표현 가능). 함수, 클래스 인스턴스, Date 이외의 복잡한 객체는 못 넘긴다 — 넘기려는 순간 에러가 나며, 이것이 "서버 코드와 클라이언트 코드는 다른 세계"라는 경계를 강제한다.

### Hydration

서버가 보낸 HTML은 아직 "죽은" 정적 문서다. 브라우저가 JS를 로드한 뒤 React가 그 HTML에 이벤트 핸들러를 붙이고 상태를 연결하는 과정이 **hydration**. 이때 서버가 그린 결과와 클라이언트가 그린 결과가 다르면 **hydration mismatch** 에러가 난다 — 대표 원인: `new Date()`나 `Math.random()`처럼 실행할 때마다 다른 값, 브라우저 전용 값(`window.innerWidth`)을 렌더링에 사용, HTML 중첩 규칙 위반(`<p>` 안에 `<div>`). 해결은 그런 값을 `useEffect` 이후에만 쓰거나 `suppressHydrationWarning`(시계 등 의도된 불일치)을 쓰는 것.

## 실습 (practice 앱에서)

- [ ] 같은 데이터(예: 현재 시각 + 외부 API 1개)를 SSG / SSR / CSR 세 방식으로 렌더링하는 페이지 3개를 만들고 비교: `npm run build` 로그의 `○`/`ƒ` 표시, 페이지 소스 보기(HTML에 데이터가 있는가?), 새로고침 시 값이 바뀌는가
  → **의미**: "어디서 + 언제"의 구분을 세 가지 관찰 방법(빌드 로그, 소스 보기, 새로고침)으로 검증한다. 00-overview의 "이 요청은 어디까지 갔다 오는가"를 내 코드로 확인하는 것.
- [ ] SSR 페이지에 `export const revalidate = 10`을 붙여 ISR로 전환, 10초 안에는 새로고침해도 같은 값, 지나면 갱신되는 것 확인
  → **의미**: SSG와 SSR 사이의 스펙트럼을 체감. 04-deployment에서 Vercel이 ISR을 어떻게 구현하는지(`x-vercel-cache` 헤더)와 연결된다.
- [ ] 서버 컴포넌트 페이지 안에 클라이언트 컴포넌트(카운터 버튼)를 심고, 반대로 카운터 안에 서버 데이터를 props로 내려보기
  → **의미**: 경계 설계의 기본기. "페이지는 서버, 잎만 클라이언트" 패턴은 이후 모든 실습(Supabase 조회 + 상호작용 UI)의 기본 형태다.
- [ ] 일부러 hydration mismatch 만들기: 서버 컴포넌트가 아닌 클라이언트 컴포넌트 렌더링에 `new Date().toISOString()`을 직접 넣고 콘솔 에러 관찰 → `useEffect`로 옮겨 해결
  → **의미**: 실무에서 반드시 만나는 에러를 통제된 환경에서 먼저 겪어본다. 에러 메시지를 아는 상태로 만나면 디버깅 시간이 크게 준다.

## 참고 자료

- https://nextjs.org/docs/app/building-your-application/rendering
- https://nextjs.org/docs/app/api-reference/functions/generate-static-params
- https://react.dev/reference/rsc/server-components (React 공식 RSC 설명)
