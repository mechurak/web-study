# 01. Postgres 기초 — 테이블 설계, SQL, 마이그레이션

> **핵심 질문**
> - 관계형 DB에서 테이블/행/열, 기본키/외래키는 무엇인가?
> - 정규화란? 언제 의도적으로 비정규화하는가?
> - 마이그레이션을 SQL 파일로 관리해야 하는 이유는? (대시보드에서 클릭으로 바꾸면 왜 위험한가)

## 개념 정리

### 테이블 설계
- [ ] 데이터 타입 선택 (`text`, `uuid`, `timestamptz`, `jsonb`…)
- [ ] 기본키 전략: `uuid` vs `bigint identity`
- [ ] 외래키와 `on delete cascade / set null`
- [ ] 인덱스 — 언제 걸고 언제 과한가

### 자주 쓰는 SQL
- [ ] SELECT / INSERT / UPDATE / DELETE, JOIN, GROUP BY
- [ ] Supabase SQL Editor에서 실행해보기

### 마이그레이션 관리
- [ ] Supabase CLI (`supabase migration new`, `supabase db push`)
- [ ] 로컬 개발 환경 (`supabase start` — 로컬 Docker Postgres)

### Supabase 특유의 것들
- [ ] `auth.users` 테이블과 내 테이블(`profiles`) 연결 패턴
- [ ] PostgREST — 테이블이 곧바로 REST API가 되는 원리

## 실습

- [ ] Supabase 프로젝트 생성, `memos` 테이블 설계 (01-nextjs 실습 앱의 메모 기능용)
- [ ] `profiles` 테이블 + `auth.users` 트리거로 자동 생성 설정
- [ ] 마이그레이션 SQL을 `practice/migrations/`에 기록

## 참고 자료

- https://supabase.com/docs/guides/database
