# 02. Supabase — DB + 인증 + 스토리지 (BaaS, Backend as a Service)

## 왜 Supabase인가

> 학습하며 아래 질문에 스스로 답을 채워나가기

- "Postgres를 서비스로" — 직접 DB 서버를 운영하는 것과 무엇이 다른가? (백업, 스케일링, 커넥션 풀)
- Firebase와 비교했을 때 장단점은? (SQL vs NoSQL, 오픈소스, 락인)
- DB 외에 인증·스토리지·실시간·Edge Functions까지 주는 것의 의미 — 초기 서비스에서 백엔드 없이 어디까지 가능한가?
- 어떤 경우에 Supabase가 안 맞는가? (복잡한 트랜잭션 로직, 특수한 인프라 요구)

## 챕터 목차

| # | 노트 | 주제 |
|---|---|---|
| 1 | [01-postgres-basics.md](./notes/01-postgres-basics.md) | 테이블 설계, SQL 기초, 마이그레이션 |
| 2 | [02-auth.md](./notes/02-auth.md) | 인증 (이메일/소셜 로그인, 세션) |
| 3 | [03-rls.md](./notes/03-rls.md) | Row Level Security — Supabase의 핵심 |
| 4 | [04-storage-realtime.md](./notes/04-storage-realtime.md) | 파일 스토리지, 실시간 구독 |
| 5 | [05-client-patterns.md](./notes/05-client-patterns.md) | Next.js 연동 패턴 (server/client/middleware) |

## 실습

01-nextjs의 practice 앱에 Supabase를 붙인다. 이 폴더의 `practice/`에는 프로젝트 설정 기록과 마이그레이션 SQL을 남긴다.
