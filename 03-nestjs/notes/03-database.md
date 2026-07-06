# 03. 데이터베이스 — Prisma/TypeORM으로 Supabase Postgres 연결

> **핵심 질문**
> - ORM(Object-Relational Mapping)은 무엇을 해주고, 무엇을 숨겨서 위험한가? (N+1, 비효율 쿼리)
> - Supabase를 "그냥 Postgres"로 쓸 수 있다 — 어떤 연결 문자열을 쓰는가? (direct vs pooler)
> - 백엔드가 DB에 직접 붙으면 RLS(Row Level Security)는 어떻게 되는가? (service 연결은 RLS 우회 → 권한 검사는 백엔드 책임)

## 개념 정리

### ORM 선택
- [ ] Prisma vs TypeORM 비교 — 이 레포에서는 하나 골라서 진행 (추천: Prisma)
- [ ] 스키마 정의 → 마이그레이션 → 타입 생성 흐름

### Supabase 연결
- [ ] Direct connection vs Connection Pooler (pgbouncer) — serverless에서 왜 pooler가 필요한가
- [ ] 기존 Supabase 마이그레이션과 ORM 마이그레이션의 공존 전략 (`prisma db pull` introspection)

### 쿼리 패턴
- [ ] CRUD(Create, Read, Update, Delete), 관계 포함 조회 (`include`/`relations`)
- [ ] 트랜잭션 — Supabase 클라이언트로는 못 하고 백엔드에서만 되는 것
- [ ] N+1 문제 재현하고 해결하기

### 책임 분리 (중요)
- [ ] Next→Supabase 직접 경로: RLS가 지킨다
- [ ] Next→Nest→Postgres 경로: RLS 우회되므로 Nest의 인가 로직이 지킨다
- [ ] 두 경로를 언제 각각 쓸지 기준 세우기

## 실습 (practice 서버에)

- [ ] Prisma 셋업, Supabase Postgres에 연결 (pooler 사용)
- [ ] 기존 `memos` 테이블 introspect해서 스키마 가져오기
- [ ] in-memory 메모 서비스를 DB 기반으로 교체
- [ ] 트랜잭션이 필요한 기능 하나 추가 (예: 메모 이동 + 카운트 갱신)

## 참고 자료

- https://docs.nestjs.com/recipes/prisma
- https://supabase.com/docs/guides/database/connecting-to-postgres
