# 03. Row Level Security (RLS) — Supabase의 핵심

> **핵심 질문**
> - 클라이언트가 DB에 직접 쿼리하는데 왜 안전한가? — 이 질문의 답이 RLS다.
> - RLS를 안 켜면 무슨 일이 일어나는가? (anon key는 공개 키다)
> - 백엔드 서버의 권한 검사 로직과 RLS는 어떻게 역할이 나뉘는가?

## 개념 정리

### 왜 RLS가 "Supabase의 핵심"인가

전통적인 구조에서 브라우저는 DB에 직접 닿지 못한다. 권한 검사는 중간의 백엔드 서버가 한다:

```
전통:     브라우저 → 백엔드 서버(여기서 "네 것 맞아?" 검사) → DB
Supabase: 브라우저 → PostgREST → DB(여기서 RLS가 검사)
```

Supabase는 그 중간 서버를 없앴다(01장 PostgREST). 그럼 검사는 누가 하나 — **DB 자신이 한다.** RLS는 Postgres의 기능으로, 테이블에 "행 단위 접근 규칙(policy)"을 선언하면 **어떤 경로로 온 쿼리든** 그 규칙을 통과한 행만 보이고/써진다. 검사가 데이터 바로 앞에 있으므로 우회 경로가 없다 — 앱 코드에 검사를 빼먹은 엔드포인트가 하나 있어도(전통 구조의 단골 사고) RLS는 뚫리지 않는다.

02장과 연결: 요청에 실린 JWT의 유저 id를 정책 안에서 `auth.uid()`로 읽는다. **Auth가 "누구인지"를 증명하고(인증), RLS가 "되는지"를 판단한다(인가).**

### RLS를 안 켜면 — anon key는 공개 키다

anon key는 브라우저 번들에 들어 있는 공개 값이다(00장 §3). 즉 **아무나** 내 프로젝트에 `GET /rest/v1/memos`를 보낼 수 있다. RLS가 꺼진 테이블은 이 요청에 전체 데이터를 돌려준다 — 읽기만이 아니라 INSERT/UPDATE/DELETE도. "anon key가 안전하다"는 말은 **RLS가 켜져 있을 때만** 참이다. 그래서 규칙은 하나: **public 스키마의 모든 테이블에 RLS를 켠다. 예외 없이.** (Supabase 대시보드도 RLS 꺼진 테이블에 경고를 띄운다.)

### policy의 구조

```sql
alter table memos enable row level security;   -- 1. 켠다 (이 순간부터 기본 전부 차단)

create policy "본인 메모만 조회"
  on memos for select
  using ( auth.uid() = user_id );              -- 2. 허용 규칙을 연다
```

- **`for select / insert / update / delete`** — 어떤 동작에 대한 규칙인지. `for all`도 가능.
- **`using (조건)`** — **이미 있는 행**에 대한 필터. select(보이는가), update/delete(대상이 되는가)에 적용.
- **`with check (조건)`** — **새로 만들어지는 행**에 대한 검사. insert와 update의 결과 행에 적용.

`using` vs `with check`가 헷갈리면 이렇게: *using = 읽을/건드릴 자격, with check = 그런 데이터를 만들 자격.* update는 둘 다 필요하다(내 행을 골라서 → 남의 것으로 바꾸는 걸 막아야 하니까).

```sql
create policy "본인 메모만 작성" on memos for insert
  with check ( auth.uid() = user_id );   -- user_id를 남의 것으로 넣는 것 차단

create policy "본인 메모만 수정" on memos for update
  using ( auth.uid() = user_id )          -- 남의 행을 고르는 것 차단
  with check ( auth.uid() = user_id );    -- 내 행을 남의 것으로 바꾸는 것 차단
```

`auth.uid()`는 요청 JWT의 `sub`(유저 id)를 돌려주는 함수다. 비로그인(anon) 요청에서는 null → 모든 비교가 false → 자동으로 차단된다.

### 대표 패턴

```sql
-- 1. 자기 것만 (개인 메모, 설정): 위의 4정책 세트

-- 2. 누구나 읽고, 주인만 쓰기 (공개 게시판)
create policy "공개 조회" on posts for select using ( true );
create policy "주인만 수정" on posts for update using ( auth.uid() = user_id );

-- 3. 역할 기반 (admin)
create policy "관리자 전체 접근" on memos for all
  using ( exists (select 1 from profiles where id = auth.uid() and is_admin) );
```

**`security definer` 함수** — 정책을 우회해야 하는 지점. 예: 3번처럼 정책이 다른 테이블을 참조하는데 그 테이블에도 RLS가 있으면 재귀 참조가 꼬일 수 있다. 이럴 때 함수 소유자 권한으로 실행되는(= RLS를 안 타는) 함수로 감싼다. 01장의 profiles 트리거가 이미 이 패턴이었다. 강력한 만큼 "이 함수 안은 무검사 구역"임을 의식하고 최소한으로.

### 백엔드 권한 검사와의 역할 분담

| | RLS | 백엔드(Server Action / NestJS) 검사 |
|---|---|---|
| 잘하는 것 | "이 행이 이 유저 것인가" (데이터 소유권) | 복잡한 비즈니스 규칙 (요금제 한도, 상태 전이, 외부 API 조건) |
| 성격 | 마지막 방어선 — 절대 뚫리면 안 되는 것 | 첫 번째 관문 — 친절한 에러, 도메인 규칙 |

둘은 대체가 아니라 **겹층**이다. 백엔드가 있어도 RLS는 켠다 — 백엔드 코드의 버그·누락이 데이터 유출로 직결되지 않게. 반대로 RLS만으로 "한 달 10개까지 작성" 같은 규칙을 짜는 것은 가능은 해도 SQL이 비대해진다 → 그런 로직이 쌓이는 것이 03-nestjs로 넘어가는 신호 중 하나.

### 함정과 디버깅

- **RLS 켰는데 policy가 없으면 전부 차단.** 그런데 에러가 아니라 **조용히 빈 배열**이 온다. "분명 데이터 있는데 안 나와요"의 1순위 용의자. select 정책부터 확인.
- **`service_role`은 RLS를 무시한다.** 관리 작업용. 서버 전용 코드에서만, 그리고 "지금 무검사로 DB를 만진다"는 자각과 함께 쓸 것.
- **정책도 쿼리다.** `using (auth.uid() = user_id)`는 모든 행에서 평가된다 — `user_id`에 인덱스가 없으면 테이블이 커질수록 느려진다(01장 인덱스). 또한 `auth.uid()`를 `(select auth.uid())`로 감싸면 행마다 재호출하지 않고 한 번만 평가돼 빨라진다 — Supabase 공식 권장 패턴.
- **디버깅 도구**: Studio SQL Editor에서 `set role authenticated; set request.jwt.claims = '{"sub":"<uuid>"}';`로 특정 유저인 척하며 쿼리해볼 수 있다. 대시보드의 Policies 화면에서 정책 목록·조건도 한눈에 확인.

## 실습

- [ ] `memos` 테이블에 RLS 적용: select/insert/update/delete 4정책으로 본인 것만 CRUD 가능하게 (마이그레이션 SQL로 작성 → `practice/migrations/`)
  → **의미**: using/with check의 구분을 손으로 익힌다. 이 정책 세트가 거의 모든 "개인 데이터" 테이블에 복붙 수준으로 재사용되는 기본형이다.
- [ ] 계정 두 개로 각각 메모 작성 → 서로의 메모가 안 보이는지, UPDATE를 시도하면 어떻게 되는지 확인
  → **의미**: "쿼리는 같은데 결과가 유저마다 다르다"를 체감한다. 앱 코드에 where user_id를 안 썼는데도 걸러진다는 것이 RLS의 본질이다.
- [ ] RLS를 잠깐 끄고 로그인 없이(anon key만으로, curl 또는 시크릿 창) 전체 테이블이 읽히는 것을 확인 → 다시 켜기
  → **의미**: "RLS 없는 Supabase = 전 세계에 공개된 DB"를 직접 본다. 한 번 보고 나면 테이블 만들 때 RLS를 빼먹는 일이 없어진다. 00장 §9 배포 체크리스트의 RLS 항목이 왜 있는지의 답.
- [ ] policy 없이 RLS만 켠 상태에서 조회 → 에러 없이 빈 배열이 오는 것 확인
  → **의미**: 최다 빈출 함정을 미리 밟아둔다. 나중에 실전에서 "데이터가 안 나올 때" 디버깅 시간을 크게 줄여준다.

## 참고 자료

- https://supabase.com/docs/guides/database/postgres/row-level-security
- https://supabase.com/docs/guides/troubleshooting/rls-performance-and-best-practices — `(select auth.uid())` 등 성능 패턴
