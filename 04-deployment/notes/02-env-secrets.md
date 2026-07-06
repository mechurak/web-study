# 02. 환경변수 & 시크릿 관리

> **핵심 질문**
> - `NEXT_PUBLIC_` 접두사의 의미는? 왜 어떤 값은 브라우저에 노출되면 안 되는가?
> - 로컬/프리뷰/프로덕션에서 서로 다른 값을 쓰는 이유와 방법은?
> - 시크릿이 유출되는 흔한 경로들은? (커밋, 클라이언트 번들, 로그)

## 개념 정리

### 분류부터
- [ ] 공개 가능 (anon key, API URL) vs 절대 비공개 (service_role, DB 비밀번호, JWT(JSON Web Token) secret)
- [ ] 우리 스택의 환경변수 전체 목록 만들기 (Next / Nest / Supabase 각각)

### Next.js + Vercel
- [ ] `.env.local` (로컬, 커밋 금지) / Vercel 대시보드 환경변수 (환경별 스코프)
- [ ] `NEXT_PUBLIC_*`는 **빌드 시점에 번들에 박힌다** — 바꾸면 재배포 필요
- [ ] `vercel env pull`로 로컬 동기화

### NestJS
- [ ] 호스팅 플랫폼의 시크릿 관리 기능 사용
- [ ] 부팅 시 검증 (03-nestjs 05장에서 만든 것 활용)

### 사고 예방
- [ ] `.gitignore`에 `.env*` 확인, `.env.example`로 필요한 키 문서화
- [ ] 유출 시 대응: 키 로테이션 절차
- [ ] 서버 로그에 시크릿 안 찍히게 주의

## 실습

- [ ] 스택 전체의 환경변수 인벤토리 표 작성 (`practice/env-inventory.md`)
- [ ] Vercel에 환경별로 다른 값 설정하고 프리뷰/프로덕션에서 값 차이 확인
- [ ] service_role 키를 실수로 `NEXT_PUBLIC_`에 넣으면 어떻게 노출되는지 번들에서 직접 확인 (로컬에서만!)

## 참고 자료

- https://vercel.com/docs/projects/environment-variables
