# 01. Postgres 기초 — 테이블 설계, SQL, 마이그레이션

> **핵심 질문**
> - 관계형 DB에서 테이블/행/열, 기본키/외래키는 무엇인가?
> - 정규화란? 언제 의도적으로 비정규화하는가?
> - 마이그레이션을 SQL 파일로 관리해야 하는 이유는? (대시보드에서 클릭으로 바꾸면 왜 위험한가)

## 개념 정리

### 관계형 모델 — 테이블/행/열, 기본키/외래키

- **테이블** = 한 종류의 데이터 집합(엑셀 시트 하나), **행(row)** = 개체 하나(메모 한 개), **열(column)** = 속성(제목, 작성일).
- **기본키(primary key)** = 행을 유일하게 식별하는 열. "이 메모"를 가리킬 수 있는 유일한 주소.
- **외래키(foreign key)** = 다른 테이블의 기본키를 참조하는 열. `memos.user_id → auth.users.id`처럼 "이 메모의 주인은 누구인가"라는 **관계**를 DB가 강제하게 만든다. 외래키가 있으면 존재하지 않는 유저의 메모를 넣으려는 INSERT가 DB 레벨에서 거부된다 — 앱 코드의 버그가 데이터를 오염시키지 못하게 하는 마지막 방어선.

**정규화**란 같은 사실을 한 곳에만 저장하는 설계다. 메모마다 작성자 이름을 문자열로 복사해 넣으면(비정규화) 유저가 이름을 바꿨을 때 모든 메모를 고쳐야 하고, 하나라도 놓치면 데이터가 서로 모순된다. 대신 `user_id`만 저장하고 이름은 `profiles`에서 JOIN으로 가져온다. **의도적 비정규화**는 읽기 성능이 증명된 병목일 때만 한다(예: 게시글마다 댓글 수를 매번 COUNT하는 대신 `comment_count` 컬럼을 두고 트리거로 갱신). 원칙: 처음엔 정규화, 비정규화는 측정 후에.

### 데이터 타입 선택 — Supabase 앱의 관례

| 용도 | 타입 | 왜 |
|---|---|---|
| 문자열 | `text` | Postgres에서 `varchar(n)`과 성능 차이 없음. 길이 제한은 필요할 때 CHECK 제약으로 |
| ID | `uuid` | 아래 기본키 전략 참고 |
| 시각 | `timestamptz` | `timestamp`(타임존 없음)를 쓰면 서버/클라이언트 타임존이 어긋나는 순간 지옥. **항상 tz 붙은 것** |
| 유연한 구조 | `jsonb` | 스키마가 정해지지 않은 부가 데이터. 단, 쿼리·제약이 필요한 필드는 컬럼으로 승격할 것 |
| 참/거짓 | `boolean` | `not null default false`로 3-상태(null) 함정 방지 |

### 기본키 전략: `uuid` vs `bigint identity`

| | `uuid` (`gen_random_uuid()`) | `bigint generated always as identity` |
|---|---|---|
| 생성 위치 | 클라이언트에서도 생성 가능 (INSERT 전에 ID를 알 수 있음) | DB만 생성 |
| 추측 가능성 | 불가 — URL에 노출해도 열거 공격 안 됨 | `/memos/1`, `/memos/2`… 순서·규모가 노출됨 |
| 크기/인덱스 | 16바이트, 랜덤이라 인덱스 지역성 나쁨 | 8바이트, 순차라 인덱스 효율 좋음 |

Supabase에서는 `auth.users.id`가 uuid이므로 **uuid로 통일**하는 것이 자연스럽다. 대규모 로그성 테이블처럼 삽입량이 극단적이면 bigint를 고려.

### 외래키의 삭제 동작 — `on delete ...`

유저가 탈퇴하면 그 유저의 메모는 어떻게 되나? 를 DB에 선언해두는 것.

- `cascade` — 부모가 지워지면 자식도 함께 삭제 (유저 삭제 → 메모 전부 삭제)
- `set null` — 자식은 남기되 참조만 끊음 (작성자 표시가 "탈퇴한 사용자"가 되는 패턴)
- 기본값(`no action`) — 자식이 있으면 부모 삭제가 **거부됨**

무엇이 맞는지는 제품 결정이다. 중요한 것은 **결정을 앱 코드가 아니라 스키마에 선언**해서, 어떤 경로로 삭제되든 일관되게 동작하게 만드는 것.

### 인덱스 — 언제 걸고 언제 과한가

인덱스는 책의 찾아보기다. `where user_id = ...`로 자주 거르는 열에 없으면 전체 테이블을 훑고(seq scan), 있으면 바로 찾아간다.

- **걸어야 하는 곳**: 외래키 컬럼(Postgres는 FK에 인덱스를 자동으로 안 만든다!), WHERE/ORDER BY에 자주 오는 컬럼, RLS 정책이 참조하는 컬럼(→ 03장 — 정책도 쿼리다).
- **과한 경우**: 모든 컬럼에 미리 걸기. 인덱스는 쓰기(INSERT/UPDATE)마다 갱신 비용을 내므로 공짜가 아니다. 원칙: FK에는 기본으로, 나머지는 느린 쿼리가 관측될 때.

### 자주 쓰는 SQL

```sql
select id, title from memos where user_id = $1 order by created_at desc limit 20;
insert into memos (user_id, title, content) values ($1, $2, $3) returning *;
update memos set title = $2, updated_at = now() where id = $1;
delete from memos where id = $1;

-- JOIN: 메모 + 작성자 이름
select m.title, p.display_name
from memos m join profiles p on p.id = m.user_id;

-- GROUP BY: 유저별 메모 수
select user_id, count(*) from memos group by user_id;
```

Supabase 클라이언트(`.from('memos').select()`)가 결국 이 SQL로 번역된다. 클라이언트 문법이 막힐 때 SQL로 생각하면 풀린다.

### 마이그레이션 관리 — 왜 SQL 파일인가

절차는 [00-setup.md §4](./00-setup.md)에 정리했다. 개념만 요약하면: **스키마의 유일한 소스는 `supabase/migrations/*.sql`이고, 대시보드 클릭은 git에 안 남는다.** 클릭으로 만든 스키마는 코드 리뷰가 불가능하고, 로컬/프로덕션 환경 재현이 안 되며, "프로덕션 DB가 코드와 다른데 뭐가 다른지 모르는" 상태를 만든다. 마이그레이션 파일이면 스키마 변경도 일반 코드처럼 PR로 다룰 수 있다.

```bash
supabase migration new create_memos   # 파일 생성
supabase db reset                     # 로컬에 적용 (처음부터 재구축)
supabase db push                      # 클라우드에 적용
```

### Supabase 특유의 것들

**`auth.users` ↔ `profiles` 패턴.** 유저 계정은 Supabase가 관리하는 `auth` 스키마의 `auth.users`에 있다. 이 테이블은 직접 컬럼을 추가하면 안 된다(Supabase 내부 소유). 그래서 앱이 관리하는 유저 정보(닉네임, 아바타)는 `public.profiles`를 따로 만들고, 가입 시 트리거로 자동 생성한다:

```sql
create table public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  display_name text,
  created_at timestamptz not null default now()
);

create function public.handle_new_user() returns trigger
language plpgsql security definer as $$
begin
  insert into public.profiles (id) values (new.id);
  return new;
end $$;

create trigger on_auth_user_created
  after insert on auth.users
  for each row execute function public.handle_new_user();
```

`security definer`인 이유: 트리거는 가입하는 유저 권한으로 도는데, 그 시점엔 아직 profiles에 쓸 권한이 없을 수 있어서 함수 소유자 권한으로 실행한다(→ 03장에서 다시).

**PostgREST — 테이블이 곧 REST API가 되는 원리.** Supabase의 `/rest/v1/*` 엔드포인트는 PostgREST라는 오픈소스 서버다. 이것이 DB 스키마를 읽어 테이블마다 REST 엔드포인트를 **자동 생성**한다: `GET /rest/v1/memos?user_id=eq.123` → `select * from memos where user_id = '123'`. 백엔드 코드를 한 줄도 안 썼는데 API가 존재하는 이유가 이것이고, "그럼 아무나 내 테이블을 읽는 것 아닌가?"라는 질문의 답이 RLS(03장)다. **이 구조를 이해하면 'Supabase = 백엔드 대체'의 실체가 보인다: CRUD API 계층을 PostgREST가, 권한 계층을 RLS가 대신하는 것.**

## 실습

- [ ] Supabase 프로젝트 생성(로컬 `supabase start` + 클라우드), `memos` 테이블 설계 — `id uuid pk`, `user_id uuid fk`, `title text`, `content text`, `created_at/updated_at timestamptz`
  → **의미**: 타입 선택·기본키 전략·외래키를 실제 결정으로 내려본다. 이 테이블이 02장(인증), 03장(RLS), 05장(CRUD 교체)까지 전 챕터의 공통 재료가 된다.
- [ ] `profiles` 테이블 + `auth.users` 트리거로 가입 시 자동 생성 설정, 실제로 가입해서 행이 생기는지 확인
  → **의미**: "Supabase 소유 스키마(auth)와 내 스키마(public)의 경계"를 몸으로 익힌다. 거의 모든 Supabase 앱이 쓰는 관용구라 한 번 만들어두면 계속 재사용한다.
- [ ] SQL Editor(Studio)에서 JOIN·GROUP BY 쿼리를 직접 실행해보고, 같은 조회를 `supabase-js`의 `.select('title, profiles(display_name)')`로도 해보기
  → **의미**: 클라이언트 문법 ↔ SQL의 대응 관계를 확인한다. PostgREST가 번역기일 뿐임을 알면 클라이언트 문법을 외우는 게 아니라 유추할 수 있게 된다.
- [ ] 위 전부를 마이그레이션 SQL로 `practice/migrations/`에 기록하고 `supabase db reset`으로 재현되는지 확인
  → **의미**: "스키마의 유일한 소스 = 마이그레이션 파일" 워크플로를 습관으로 만든다. reset이 통과하면 어떤 환경에서도 같은 스키마를 재현할 수 있다는 증명이다.

## 참고 자료

- https://supabase.com/docs/guides/database
- https://supabase.com/docs/guides/auth/managing-user-data — profiles 트리거 패턴 공식 문서
- https://postgrest.org/en/stable/ — 테이블→API 자동 생성의 원리
