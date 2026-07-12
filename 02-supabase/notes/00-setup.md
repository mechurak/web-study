# 00. 세팅 가이드 — 로컬 개발부터 Vercel + Supabase 프로덕션까지

> **핵심 질문**
> - 로컬 Supabase와 클라우드 Supabase는 어떤 관계인가? 스키마는 어떻게 둘 사이에서 동기화되는가?
> - anon key는 왜 브라우저에 노출해도 되고, service_role key는 왜 절대 안 되는가?
> - 구글 OAuth의 redirect URI는 왜 내 앱 주소가 아니라 `supabase.co` 주소인가?

이 문서는 개념 노트가 아니라 **절차 가이드**다. 새 프로젝트에 Vercel + Supabase를 붙일 때 위에서부터 순서대로 따라가면 된다. 실전 적용 예시는 `~/workspace/time-table/docs/setup/m4-supabase.md` (M4 세팅 기록) 참고.

## 0. 큰 그림 — 두 개의 Supabase, 하나의 스키마 소스

세팅의 전체 구조를 먼저 이해해야 각 스텝이 왜 필요한지 보인다.

```
로컬 머신                          클라우드
┌─────────────────────┐          ┌──────────────────────┐
│ supabase start       │          │ supabase.com 프로젝트  │
│ (Docker로 Postgres·  │          │ (실제 서비스용 DB·     │
│  Auth·Storage 재현)  │          │  Auth·Storage)        │
└─────────▲───────────┘          └──────────▲───────────┘
          │ supabase db reset                │ supabase db push
          └──────── supabase/migrations/*.sql ┘
                    (스키마의 유일한 소스, git으로 관리)
```

- **로컬 스택**: Supabase CLI(Command Line Interface)가 Docker 컨테이너로 Postgres + Auth + Storage + Studio(관리 UI)를 통째로 띄운다. 개발 중에는 이걸 쓴다.
- **클라우드 프로젝트**: supabase.com에서 만드는 실제 서비스용 인스턴스. Vercel에 배포된 앱이 여기에 붙는다.
- **마이그레이션 파일**이 둘을 잇는다. 스키마(테이블, RLS 정책, 트리거)는 반드시 `supabase/migrations/*.sql`에 SQL로 남기고, 로컬엔 `db reset`으로, 클라우드엔 `db push`로 적용한다.

**왜 대시보드에서 테이블을 직접 만들면 안 되나?** 클릭으로 만든 스키마는 git에 안 남는다 → 코드 리뷰 불가, 다른 환경(로컬/스테이징) 재현 불가, "프로덕션 DB가 코드와 다른데 뭐가 다른지 모르는" 상태가 된다. 마이그레이션 파일을 유일한 소스로 삼으면 스키마 변경도 코드 변경과 똑같이 PR로 다룰 수 있다.

## 1. 사전 준비

```bash
brew install supabase/tap/supabase   # Supabase CLI
```

Docker 런타임이 필요하다(로컬 스택이 전부 컨테이너라서). Docker Desktop 또는 colima 등 아무거나. colima를 쓴다면 §2의 알려진 이슈 참고.

## 2. 프로젝트 초기화와 로컬 스택

앱 저장소 루트에서:

```bash
supabase init      # supabase/ 폴더 생성 (config.toml + migrations/)
supabase start     # 로컬 스택 기동. 첫 실행은 이미지 다운로드로 몇 분 걸림
supabase status    # URL과 키 확인
```

`supabase status`가 보여주는 것 중 당장 쓰는 값:

| 값 | 용도 |
|---|---|
| `API URL` | 앱의 `NEXT_PUBLIC_SUPABASE_URL` |
| `anon key` | 앱의 `NEXT_PUBLIC_SUPABASE_ANON_KEY` |
| `service_role key` | 로컬 테스트 유저 생성 등 관리 작업(§5) |
| `Studio URL` | 브라우저 관리 UI — 테이블/데이터/로그 확인 |

스택 중지는 `supabase stop`(데이터 유지), 초기화는 `supabase db reset`(마이그레이션 처음부터 재적용 — 스키마 꼬였을 때 리셋 버튼).

### config.toml에서 알아둘 것

- **포트 충돌**: 로컬 Supabase 프로젝트를 두 개 이상 돌리면 기본 포트(54321 등)가 겹친다. `config.toml`에서 프로젝트별로 포트 대역을 옮겨두면 동시에 띄울 수 있다. (time-table은 `5532x` 대역으로 옮겨서 API가 `55321`.)
- **colima 알려진 이슈**: 컨테이너 마운트/헬스체크 문제로 `realtime`, `analytics`, `storage.image_transformation`이 안 뜨는 경우가 있다. 안 쓰는 기능이면 `config.toml`에서 `enabled = false`로 끄면 된다. Docker Desktop에서는 문제없다.

## 3. 환경변수 — 키 3종의 역할과 왜 공개 여부가 갈리는가

| 변수 | 정체 | 공개 가능? |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | 프로젝트 API 엔드포인트 | O |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | 익명(공개) 키 | O — 브라우저에 노출됨 |
| `SUPABASE_SERVICE_ROLE_KEY` | 관리자 키, **RLS(Row Level Security) 우회** | **절대 금지** |

**anon key가 브라우저에 노출돼도 되는 이유**: 이 키는 "권한"이 아니라 "어느 프로젝트인지 식별 + RLS 적용 대상임"을 나타낸다. 실제 접근 제어는 키가 아니라 **RLS 정책**이 한다(→ [03-rls.md](./03-rls.md)). 뒤집어 말하면, **RLS를 안 켠 테이블은 anon key만으로 전 세계에서 읽고 쓸 수 있다**. anon key 공개가 안전한 것은 RLS가 전제일 때만이다.

**service_role key가 절대 노출되면 안 되는 이유**: RLS를 통째로 우회한다. 유출되면 DB 전체를 누구나 읽고 쓸 수 있다. `NEXT_PUBLIC_` 접두사를 붙이는 순간 Next.js가 브라우저 번들에 값을 인라인하므로, 이 키에는 절대 붙이지 말고 서버 전용 코드(Route Handler, Server Action)에서만 읽는다.

> 참고: Supabase가 2025년부터 새 키 체계(`sb_publishable_...` / `sb_secret_...`)로 전환 중이다. 역할은 각각 anon / service_role과 같다고 보면 된다. 새 프로젝트 대시보드에 이 이름으로 보여도 당황하지 말 것.

파일 배치:

```bash
# .env.local  (gitignore 필수 — 로컬 개발값)
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321   # supabase status의 API URL
NEXT_PUBLIC_SUPABASE_ANON_KEY=<supabase status의 anon key>
```

`.env.example`에 키 이름만(값 없이) 커밋해두면 다른 환경에서 뭘 채워야 하는지 문서가 된다.

## 4. 스키마 작업 루프 (로컬)

```bash
supabase migration new create_memos   # supabase/migrations/<timestamp>_create_memos.sql 생성
# → SQL 작성 (CREATE TABLE + RLS 정책까지 한 파일에)
supabase db reset                     # 로컬 DB를 마이그레이션으로 재구축
supabase gen types typescript --local > lib/database.types.ts   # TS 타입 재생성
```

- 마이그레이션 파일에 **테이블과 RLS 정책을 함께** 넣는다. "테이블은 있는데 정책을 깜빡한" 창이 생기지 않게.
- Studio에서 GUI로 만져보고 `supabase db diff -f <name>`으로 변경분을 마이그레이션으로 뽑아내는 역방향 워크플로도 가능하다. 단, 최종 소스는 항상 마이그레이션 파일.
- `gen types`는 스키마가 바뀔 때마다 다시 돌린다. 쿼리 결과 타입이 DB와 어긋난 채 컴파일이 통과하는 걸 막아준다.

## 5. 로컬에서 로그인 개발하기

이메일/비밀번호 로그인은 로컬 스택에서 그대로 동작한다(확인 메일은 실제 발송 대신 로컬 메일 캐처 **Inbucket**에 잡힌다 — `supabase status`에 URL 있음).

**소셜 로그인(구글 등)은 로컬에서 바로 안 된다** — OAuth 앱에 등록된 redirect URI가 클라우드 주소라서. 두 가지 우회:

1. (간단) 로컬에서는 테스트 유저를 admin API로 만들어 이메일 로그인으로 개발:

   ```js
   import { createClient } from '@supabase/supabase-js'
   const admin = createClient(URL, SERVICE_ROLE_KEY, { auth: { persistSession: false } })
   await admin.auth.admin.createUser({
     email: 'me@test.dev', password: 'password123', email_confirm: true,
   })
   ```

2. (정석) `config.toml`의 `[auth.external.google]`에 client_id/secret을 넣으면 로컬 스택에서도 구글 로그인이 된다. 구글 콘솔의 redirect URI에 `http://127.0.0.1:54321/auth/v1/callback`(로컬 API 포트) 추가 필요.

## 6. 클라우드 프로젝트 생성과 스키마 push

1. https://supabase.com → New Project. **리전은 사용자와 가까운 곳**(예: 한국 대상이면 서울/도쿄) — DB까지의 왕복 지연이 모든 쿼리에 더해지므로 나중에 못 바꾸는 것 중 가장 체감 큰 선택이다. DB 비밀번호는 비밀번호 관리자에 보관(대시보드에서 다시 못 봄, 리셋만 가능).
2. 로컬 저장소를 클라우드 프로젝트에 연결하고 마이그레이션 적용:

   ```bash
   supabase link --project-ref <project-ref>   # ref는 대시보드 URL에 있음
   supabase db push                            # 미적용 마이그레이션을 원격에 실행
   ```

   **왜 대시보드 SQL Editor가 아니라 push인가**: 로컬에서 검증한 것과 *동일한 파일*이 적용되므로 로컬 = 프로덕션 스키마가 보장된다. 손으로 붙여넣으면 언젠가 어긋난다.
3. 대시보드 → Project Settings → API에서 **Project URL**과 **anon key**를 복사 → Vercel 환경변수로(§8).

## 7. 구글 OAuth 세팅

먼저 흐름을 이해하면 각 설정값이 자명해진다:

```
브라우저 → 구글 로그인 → ①(redirect URI) Supabase Auth 서버 → ②(Redirect URLs) 내 앱 /auth/callback
```

로그인의 상대방은 내 앱이 아니라 **Supabase Auth 서버**다. 구글은 Supabase로 돌려보내고(①), Supabase가 세션 코드를 들고 내 앱으로 돌려보낸다(②). 그래서 설정이 두 군데로 갈린다.

### 7.1 Google Cloud Console (① 설정)

1. https://console.cloud.google.com → 프로젝트 생성/선택.
2. APIs & Services → OAuth consent screen 구성 (External, 앱 이름, 지원 이메일).
3. Credentials → Create Credentials → OAuth client ID → Web application.
4. **Authorized redirect URIs** = Supabase 콜백 (내 앱 주소가 아님!):

   ```
   https://<project-ref>.supabase.co/auth/v1/callback
   ```
5. Client ID / Client Secret 복사.

### 7.2 Supabase 대시보드 (② 설정)

1. Authentication → Providers → **Google** 활성화, 위의 Client ID/Secret 입력.
2. Authentication → URL Configuration:
   - **Site URL**: 프로덕션 도메인 (예: `https://myapp.vercel.app`). 리다이렉트 목적지의 기본값.
   - **Redirect URLs**: Supabase가 로그인 후 돌려보내줄 수 있는 주소의 **허용 목록**:

     ```
     https://<프로덕션 도메인>/auth/callback
     http://localhost:3000/auth/callback
     https://*-<vercel팀슬러그>.vercel.app/auth/callback   ← Vercel preview 배포에서도 로그인하려면 (와일드카드 지원)
     ```

   **왜 허용 목록인가**: `redirectTo`를 아무 주소나 받아주면 공격자가 로그인 코드를 자기 사이트로 빼돌리는 open redirect가 된다. 여기 없는 주소로는 Supabase가 리다이렉트를 거부한다(대신 Site URL로 보냄 — "로그인은 됐는데 엉뚱한 주소로 떨어진다"면 십중팔구 이 목록 누락).

앱 쪽 코드는 `signInWithOAuth({ provider: 'google', redirectTo: <origin>/auth/callback })` 호출 + `/auth/callback` Route Handler에서 code→세션 교환(→ [02-auth.md](./02-auth.md), [05-client-patterns.md](./05-client-patterns.md)).

## 8. Vercel 연동

1. Vercel 프로젝트 → Settings → Environment Variables에 `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` 추가. **Production과 Preview 둘 다** 체크 — preview 배포도 같은 Supabase를 바라보게 하려면 필요하다(스테이징 DB를 따로 두는 팀은 Preview에 다른 값을 넣는다).
2. `NEXT_PUBLIC_` 값은 **빌드 시점에** 번들에 박히므로, 환경변수를 바꾸면 재배포(Redeploy)해야 반영된다. 값만 저장하고 "왜 안 바뀌지" 하는 게 단골 함정.
3. service_role key를 서버에서 쓰는 단계가 오면 그때 `SUPABASE_SERVICE_ROLE_KEY`로 추가한다(Production만, `NEXT_PUBLIC_` 금지).

## 9. 배포 후 확인 체크리스트

- [ ] 홈이 뜨고 콘솔에 Supabase 연결 에러가 없는가 (URL/키 오타 검출)
- [ ] 회원가입/로그인 → 로그아웃 → 재로그인이 되는가
- [ ] 구글 로그인 → 콜백 후 **의도한 페이지**로 돌아오는가 (Redirect URLs 검증)
- [ ] 데이터 생성 후 **다른 브라우저에서 남의 계정으로 로그인** → 그 데이터가 안 보이는가 (RLS 동작 검증 — 이걸 안 하면 RLS 누락을 프로덕션에서 사용자가 발견한다)
- [ ] Storage를 쓴다면: 업로드 → URL 접근 → 새로고침 후 유지되는가

## 참고 자료

- https://supabase.com/docs/guides/local-development — 로컬 개발 공식 가이드
- https://supabase.com/docs/guides/deployment/database-migrations — 마이그레이션 워크플로
- https://supabase.com/docs/guides/auth/social-login/auth-google — 구글 로그인
- `~/workspace/time-table/docs/setup/m4-supabase.md` — 이 절차를 실제 프로젝트에 적용한 기록
