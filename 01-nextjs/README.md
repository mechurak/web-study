# 01. Next.js — Frontend (+ 가벼운 백엔드)

## 왜 Next.js인가

> 학습하며 아래 질문에 스스로 답을 채워나가기

- 순수 React SPA(Vite 등)와 비교해 무엇을 더 해주는가? (라우팅, SSR, 번들 최적화, 이미지 처리…)
- SEO와 첫 로딩 성능이 왜 SPA의 약점이고, Next.js는 어떻게 해결하는가?
- 서버 컴포넌트(RSC)가 등장한 배경은? 무엇이 클라이언트로 안 내려가서 좋은가?
- 어떤 경우에 Next.js가 **과한** 선택인가? (관리자 페이지, 내부 도구 등)

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
