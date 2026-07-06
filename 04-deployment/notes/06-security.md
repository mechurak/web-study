# 06. 웹 보안 기초 — 공격자 관점으로 스택 점검하기

> **핵심 질문**
> - XSS와 CSRF는 각각 무엇을 속이는 공격인가? (브라우저를 속이기 vs 서버를 속이기)
> - React/Next.js가 기본으로 막아주는 것과 안 막아주는 것은?
> - 우리 스택(Next + Supabase + Nest)에서 각 레이어의 보안 책임은 무엇인가?

## 개념 정리

### XSS (Cross-Site Scripting)
- [ ] 저장형/반사형 XSS 시나리오
- [ ] React가 기본 이스케이프해주는 것, 뚫리는 곳: `dangerouslySetInnerHTML`, `href="javascript:"`, 마크다운 렌더링
- [ ] CSP(Content-Security-Policy) 헤더 — next.config에서 설정

### CSRF & 인증 쿠키
- [ ] CSRF 시나리오와 `SameSite` 쿠키의 방어
- [ ] Server Action의 origin 검증 (Next가 해주는 것)
- [ ] JWT를 localStorage에 두면 안 되는 이유 (XSS와의 조합)

### 주입 (Injection)
- [ ] SQL Injection — ORM/파라미터 바인딩이 지켜주는 것, 뚫리는 곳 (raw query 문자열 조립)
- [ ] 경로 조작, 커맨드 주입 맛보기

### 우리 스택의 책임 지도
- [ ] Supabase 직접 경로: RLS가 최후 방어선 (02-supabase 03장) + anon key는 공개 전제
- [ ] Nest 경로: JWT 검증(Guard) + 소유권 검사 + 입력 검증(DTO/zod)
- [ ] 시크릿 관리 (이 챕터 02장), CORS 최소 허용 (03장)
- [ ] Rate limiting — Vercel/Nest(`@nestjs/throttler`)에서 남용 방어
- [ ] 의존성 취약점: `npm audit`, Dependabot

### OWASP Top 10
- [ ] 항목별로 "우리 스택에서는 어디에 해당? 무엇으로 방어?" 표 만들기

## 실습

- [ ] 메모 내용에 `<script>` 삽입해보기 — React가 막는 것 확인 후, `dangerouslySetInnerHTML`로 일부러 뚫어보고 되돌리기
- [ ] 보안 헤더(CSP 등) 설정하고 securityheaders.com 으로 채점
- [ ] RLS 끈 상태 + anon key로 데이터가 새는 것 재확인 (02-supabase 03장 실습과 연결)
- [ ] Nest API에 rate limit 걸고 연속 요청으로 429 확인
- [ ] 배포 전 보안 체크리스트를 `practice/checklist.md`에 정리

## 참고 자료

- https://owasp.org/www-project-top-ten/
- https://nextjs.org/docs/app/building-your-application/configuring/content-security-policy
