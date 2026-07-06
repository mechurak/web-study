# 04. 인증/인가 — Guard, JWT, Supabase 토큰 검증

> **핵심 질문**
> - 프론트는 Supabase로 로그인하는데, NestJS는 그 유저를 어떻게 믿는가? (JWT(JSON Web Token) 검증)
> - Guard는 요청 생명주기의 어디에서 실행되고, 무엇을 반환하는가?
> - 인가(내 메모인지 확인)는 Guard와 서비스 중 어디에서 하는가?

## 개념 정리

### 아키텍처: Supabase 인증 + Nest 검증
- [ ] 흐름: 브라우저(Supabase 로그인, JWT 발급) → Nest API 호출 시 `Authorization: Bearer` → Nest가 JWT 서명 검증
- [ ] Supabase JWT Secret(또는 JWKS)으로 토큰 검증하기
- [ ] 이 구조의 장점: Nest에 비밀번호/세션 저장 로직이 없어도 됨

### NestJS 구성 요소
- [ ] AuthGuard 만들기 (`CanActivate`)
- [ ] `@UseGuards()` — 컨트롤러/핸들러 단위 적용, 전역 적용 + `@Public()` 예외
- [ ] 커스텀 데코레이터 `@CurrentUser()`로 핸들러에서 유저 꺼내기

### 인가 (Authorization)
- [ ] 리소스 소유권 검사 — 서비스 레이어에서 (예: 남의 메모 수정 시 403)
- [ ] 역할 기반 접근 제어 (RolesGuard) 맛보기

## 실습 (practice 서버에)

- [ ] Supabase JWT를 검증하는 AuthGuard 구현
- [ ] 메모 API 전체에 인증 적용, 본인 메모만 수정/삭제 가능하게
- [ ] Next.js에서 세션 토큰을 실어 Nest API 호출하는 유틸 작성
- [ ] 만료된/변조된 토큰으로 401 떨어지는 것 확인

## 참고 자료

- https://docs.nestjs.com/guards
- https://supabase.com/docs/guides/auth/jwts
