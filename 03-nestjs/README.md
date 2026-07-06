# 03. NestJS — 본격 Backend API 서버

## 왜 NestJS인가

> 학습하며 아래 질문에 스스로 답을 채워나가기

- Next.js API Route + Supabase만으로 충분하지 않은 순간은 언제 오는가?
  (복잡한 비즈니스 로직, 트랜잭션, 배치/크론, 웹소켓, 외부 시스템 연동, 팀 규모…)
- Express와 비교해 NestJS가 강제하는 구조(모듈, DI(Dependency Injection, 의존성 주입), 데코레이터)는 무엇을 좋게 만드는가?
- "프레임워크가 구조를 강제한다"의 장단점 — 어떤 팀/프로젝트에 맞는가?
- Spring과 닮은 점 — 백엔드 설계 관례(레이어드 아키텍처)가 왜 언어를 넘어 비슷한가?

## 챕터 목차

| # | 노트 | 주제 |
|---|---|---|
| 1 | [01-architecture.md](./notes/01-architecture.md) | 모듈/컨트롤러/프로바이더, DI |
| 2 | [02-rest-api.md](./notes/02-rest-api.md) | DTO(Data Transfer Object), 검증(Pipe), 예외 처리 |
| 3 | [03-database.md](./notes/03-database.md) | Prisma/TypeORM으로 Supabase Postgres 연결 |
| 4 | [04-auth.md](./notes/04-auth.md) | Guard, JWT(JSON Web Token), Supabase 토큰 검증 |
| 5 | [05-testing-config.md](./notes/05-testing-config.md) | 테스트, 환경변수/설정 관리 |

## 실습

`practice/`에 NestJS API 서버를 만든다. 01-nextjs 앱의 메모 기능 중 "복잡한 로직이 필요한 부분"을 이 서버로 옮겨오며, Next(프론트) ↔ Nest(백엔드) ↔ Supabase(DB) 구조를 완성한다.
