# 01. Next.js — Frontend (+ 가벼운 백엔드)

## 왜 Next.js인가

**React는 라이브러리고, Next.js는 프레임워크다.** React 자체는 "UI를 컴포넌트로 그리는 방법"만 제공한다. 실제 서비스에 필요한 나머지 — 라우팅, 서버 렌더링, 코드 분할, 이미지/폰트 최적화, API 엔드포인트, 빌드/배포 파이프라인 — 를 Next.js가 하나의 관례로 묶어준다. Vite + React로 시작하면 이것들을 라이브러리 조합으로 직접 결정해야 한다.

**SPA의 약점을 서버 렌더링으로 해결한다.** 순수 SPA는 빈 HTML + 큰 JS 번들을 내려보내고 브라우저에서 화면을 그린다. 그래서 (1) 첫 화면이 늦고(JS 다 받아야 보임), (2) 검색 엔진 크롤러가 빈 문서를 본다(SEO 약함). Next.js는 서버에서 완성된 HTML을 먼저 보내고(00-overview의 "요청의 일생" ⑤~⑦), JS는 나중에 붙인다(hydration). 첫 화면과 SEO를 모두 잡는 구조다. → 자세히는 [02-rendering.md](./notes/02-rendering.md)

**서버 컴포넌트(RSC)는 "클라이언트로 안 보내는 코드"를 만든다.** 기존에는 모든 컴포넌트 코드가 JS 번들에 포함돼 브라우저로 갔다. RSC는 서버에서만 실행되고 결과만 내려보내므로 — DB 접근 코드, 무거운 라이브러리, 시크릿이 번들에서 빠진다. 번들은 작아지고, 데이터 접근은 가까워진다(컴포넌트 안에서 바로 `await db.query()`). 이것이 App Router 설계의 중심이다.

**Next.js가 과한 경우도 안다.** 로그인 뒤에서만 쓰는 관리자 페이지·사내 도구는 SEO도 첫 로딩도 덜 중요하다 — Vite SPA가 더 단순하고 빠르게 만든다. 서버 인프라 없이 정적 호스팅만 가능한 환경도 마찬가지. "공개 서비스면 Next, 내부 도구면 SPA도 충분"이 실무 감각에 가깝다. (00-overview의 비교표 참고)

## 챕터 목차

| # | 노트 | 주제 |
|---|---|---|
| 1 | [01-routing.md](./notes/01-routing.md) | App Router, 레이아웃, 동적 라우트 |
| 2 | [02-rendering.md](./notes/02-rendering.md) | SSR/SSG/ISR/CSR, 서버 컴포넌트 |
| 3 | [03-data-fetching.md](./notes/03-data-fetching.md) | fetch 캐싱, Server Actions |
| 4 | [04-api-routes.md](./notes/04-api-routes.md) | Route Handlers, 미들웨어 |
| 5 | [05-optimization.md](./notes/05-optimization.md) | 이미지/폰트 최적화, 메타데이터(SEO) |
| 6 | [06-tailwind.md](./notes/06-tailwind.md) | Tailwind CSS — 유틸리티 퍼스트 스타일링 |
| 7 | [07-shadcn.md](./notes/07-shadcn.md) | shadcn/ui — 복사해서 소유하는 컴포넌트 |
| 8 | [08-state-management.md](./notes/08-state-management.md) | 서버 상태(TanStack Query) vs 클라이언트 상태(Zustand) |
| 9 | [09-zod.md](./notes/09-zod.md) | zod — 스키마 검증 + 타입 추론 |
| 10 | [10-testing.md](./notes/10-testing.md) | Playwright E2E, React Testing Library |

## 실습

`practice/`에 Next.js 앱 하나를 만들고, 각 챕터의 개념을 그 앱 안의 페이지/기능으로 확장해나간다.
(챕터마다 새 앱을 만들지 않는다 — [practice/README.md](./practice/README.md) 참고)
