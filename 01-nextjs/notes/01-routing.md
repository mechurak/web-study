# 01. 라우팅 — App Router

> **핵심 질문**
> - 파일시스템 기반 라우팅이란? `app/` 폴더 구조가 URL과 어떻게 대응되는가?
> - `layout.tsx`는 `page.tsx`와 무엇이 다르고, 왜 리렌더링되지 않는가?
> - Pages Router와 App Router의 차이는? 왜 App Router로 넘어갔는가?

## 개념 정리

### 기본 라우팅
- [ ] `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`의 역할
- [ ] 중첩 레이아웃 (nested layouts)

### 동적 라우트
- [ ] `[slug]`, `[...slug]`, `[[...slug]]`의 차이
- [ ] `params` 받아서 사용하기

### 네비게이션
- [ ] `<Link>` vs `<a>` — 무엇이 다른가 (prefetch, 클라이언트 전환)
- [ ] `useRouter`, `redirect`, `usePathname`

### 고급
- [ ] Route Groups `(group)` — URL에 영향 없이 구조화
- [ ] Parallel Routes `@slot`, Intercepting Routes `(.)` — 언제 쓰나

## 실습 (practice 앱에서)

- [ ] 홈 / 목록 / 상세(`[id]`) 3단 구조 페이지 만들기
- [ ] 공통 네비게이션 바를 루트 레이아웃에 배치
- [ ] `loading.tsx`로 로딩 UI, 존재하지 않는 id에 `notFound()` 처리

## 참고 자료

- https://nextjs.org/docs/app/building-your-application/routing
