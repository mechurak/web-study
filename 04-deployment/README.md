# 04. 배포 & 운영 — Vercel 그리고 그 너머

## 배포의 전체 그림

> 학습하며 아래 질문에 스스로 답을 채워나가기

- 우리 스택의 각 조각은 어디에 올라가는가?
  - Next.js → **Vercel**
  - NestJS → Vercel이 아닌 곳 (Railway / Fly.io / Cloud Run — 왜?)
  - Postgres/인증/스토리지 → **Supabase** (이미 호스팅됨)
- "배포한다"는 것은 정확히 무슨 과정인가? (빌드 → 아티팩트 → 서빙 인프라 → 도메인/TLS)
- serverless와 상시 서버(long-running)의 근본적 차이 — 무엇이 각 플랫폼의 적합성을 가르는가?

## 챕터 목차

| # | 노트 | 주제 |
|---|---|---|
| 1 | [01-vercel.md](./notes/01-vercel.md) | Vercel 동작 원리 (serverless/edge), 프리뷰 배포 |
| 2 | [02-env-secrets.md](./notes/02-env-secrets.md) | 환경변수·시크릿 관리 (로컬/프리뷰/프로덕션) |
| 3 | [03-nestjs-hosting.md](./notes/03-nestjs-hosting.md) | NestJS 호스팅 선택지 비교와 배포 |
| 4 | [04-domain-ci.md](./notes/04-domain-ci.md) | 커스텀 도메인, GitHub 연동 CI/CD |
| 5 | [05-monitoring.md](./notes/05-monitoring.md) | 로그, 애널리틱스, 에러 추적 |
| 6 | [06-security.md](./notes/06-security.md) | 웹 보안 기초 — XSS/CSRF/주입, OWASP Top 10 |

## 실습

새 코드를 만들지 않는다. **01~03에서 만든 실습 앱들을 실제로 배포**하는 것이 실습이다.
`practice/`에 배포 체크리스트와 설정 기록을 남긴다.
