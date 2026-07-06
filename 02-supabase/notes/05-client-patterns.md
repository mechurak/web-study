# 05. Next.js 연동 패턴 — server / client / middleware

> **핵심 질문**
> - 왜 Supabase 클라이언트를 서버용/브라우저용으로 따로 만들어야 하는가?
> - 서버 컴포넌트, 클라이언트 컴포넌트, Server Action, Route Handler, 미들웨어 — 각각 어떤 클라이언트를 쓰는가?
> - 세션 갱신(refresh)은 어디서 일어나야 하는가? (미들웨어의 역할)

## 개념 정리

### `@supabase/ssr` 3종 클라이언트
- [ ] `createBrowserClient` — 클라이언트 컴포넌트용
- [ ] `createServerClient` — 서버 컴포넌트/Server Action/Route Handler용 (쿠키 읽기·쓰기)
- [ ] 미들웨어에서의 세션 갱신 패턴 (`updateSession`)

### 폴더 구조 관례
- [ ] `lib/supabase/client.ts`, `lib/supabase/server.ts`, `lib/supabase/middleware.ts`

### 데이터 페칭 위치 결정
- [ ] 초기 데이터: 서버 컴포넌트에서 조회 → props로 전달
- [ ] 변경(mutation): Server Action에서 처리 + `revalidatePath`
- [ ] 실시간/인터랙티브: 클라이언트 컴포넌트에서 browser client

### 타입 안전성
- [ ] `supabase gen types typescript` — DB 스키마에서 TS 타입 생성

## 실습 (01-nextjs practice 앱에)

- [ ] 3종 클라이언트 셋업을 관례 구조로 정리
- [ ] 메모 CRUD를 in-memory → Supabase로 전면 교체 (조회는 서버 컴포넌트, 변경은 Server Action)
- [ ] 생성된 타입으로 쿼리 결과 타입 안전하게 만들기

## 참고 자료

- https://supabase.com/docs/guides/auth/server-side/nextjs
