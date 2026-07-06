# 04. API Routes — Route Handlers, 미들웨어

> **핵심 질문**
> - Next.js 안에서 백엔드 API를 만들 수 있는데, 왜 NestJS 같은 별도 백엔드가 필요해지는가?
> - Route Handler는 어떤 런타임에서 실행되는가? (Node vs Edge)
> - 미들웨어로 할 수 있는 것과 하면 안 되는 것은?

## 개념 정리

### Route Handlers
- [ ] `app/api/*/route.ts` — GET/POST/PUT/DELETE 핸들러 작성법
- [ ] `NextRequest` / `NextResponse`, 쿼리·바디 파싱
- [ ] 동적 세그먼트 (`app/api/posts/[id]/route.ts`)

### 미들웨어
- [ ] `middleware.ts` — 모든 요청 전에 실행 (리다이렉트, 헤더, 인증 체크)
- [ ] `matcher`로 적용 경로 제한
- [ ] Edge 런타임 제약 (Node API 못 씀)

### Next API의 한계 — NestJS가 필요해지는 지점
- [ ] 복잡한 비즈니스 로직의 구조화 (모듈/DI 없음)
- [ ] 장시간 작업, 웹소켓, 크론, 큐 — serverless와 안 맞는 것들
- [ ] 프론트 배포와 백엔드 배포가 묶이는 문제

## 실습 (practice 앱에서)

- [ ] 메모 CRUD를 Route Handler로 구현 (`GET/POST /api/memos`, `DELETE /api/memos/[id]`)
- [ ] 미들웨어로 특정 경로 접근 시 로그인 페이지로 리다이렉트 (인증은 02-supabase에서 실제 구현)

## 참고 자료

- https://nextjs.org/docs/app/building-your-application/routing/route-handlers
