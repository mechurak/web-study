# 03. Row Level Security (RLS) — Supabase의 핵심

> **핵심 질문**
> - 클라이언트가 DB에 직접 쿼리하는데 왜 안전한가? — 이 질문의 답이 RLS다.
> - RLS를 안 켜면 무슨 일이 일어나는가? (anon key는 공개 키다)
> - 백엔드 서버의 권한 검사 로직과 RLS는 어떻게 역할이 나뉘는가?

## 개념 정리

### RLS 기본
- [ ] `alter table ... enable row level security`
- [ ] policy 구조: `for select/insert/update/delete`, `using` vs `with check`
- [ ] `auth.uid()` — 정책 안에서 현재 유저 알아내기

### 대표 패턴
- [ ] "자기 것만 읽고 쓰기": `using (auth.uid() = user_id)`
- [ ] "누구나 읽고, 주인만 쓰기" (공개 게시판)
- [ ] 역할 기반 (admin 컬럼/테이블 참조 정책)
- [ ] `security definer` 함수로 정책 우회가 필요한 경우

### 함정과 디버깅
- [ ] RLS 켰는데 policy가 없으면? → 전부 차단 (조용히 빈 배열 반환되는 함정)
- [ ] `service_role`은 RLS를 무시한다 — 서버에서만 쓸 것
- [ ] 정책 성능: 정책도 쿼리다 (인덱스 필요)

## 실습

- [ ] `memos` 테이블에 RLS 적용: 본인 메모만 CRUD 가능하게
- [ ] 다른 계정으로 로그인해서 남의 메모가 안 보이는지 확인
- [ ] RLS 끈 상태에서 anon key로 전체 테이블이 읽히는 것을 직접 확인 (왜 필수인지 체감)

## 참고 자료

- https://supabase.com/docs/guides/database/postgres/row-level-security
