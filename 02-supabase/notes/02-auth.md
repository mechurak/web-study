# 02. 인증 — 이메일/소셜 로그인, 세션

> **핵심 질문**
> - 인증(Authentication)과 인가(Authorization)의 차이는?
> - JWT(JSON Web Token) 기반 세션은 어떻게 동작하는가? access token / refresh token의 역할은?
> - 서버 컴포넌트에서 "현재 로그인한 유저"를 어떻게 아는가? (쿠키 기반 세션)

## 개념 정리

### 인증 방식
- [ ] 이메일+비밀번호, 매직 링크, OTP(One-Time Password)
- [ ] 소셜 로그인 (OAuth) — Google/GitHub 설정 흐름
- [ ] 리다이렉트 URL, 콜백 처리 (`/auth/callback`)

### 세션 관리
- [ ] JWT 구조 (header/payload/signature), 만료와 갱신
- [ ] 브라우저(localStorage vs cookie)와 서버에서의 세션 저장 위치
- [ ] `@supabase/ssr` — Next.js에서 쿠키 기반 세션이 필요한 이유

### 보안 기본기
- [ ] `anon key` vs `service_role key` — 각각 어디에 두어야 하는가 (service_role은 절대 클라이언트 금지)
- [ ] 이메일 확인, 비밀번호 정책, Rate limit

## 실습 (01-nextjs practice 앱에)

- [ ] 이메일 회원가입/로그인/로그아웃 구현
- [ ] GitHub OAuth 로그인 추가
- [ ] 로그인 상태에 따라 네비게이션 바 UI 분기
- [ ] 미들웨어에서 미로그인 시 보호 페이지 접근 차단 (01-nextjs 04장의 미들웨어 실습 완성)

## 참고 자료

- https://supabase.com/docs/guides/auth
