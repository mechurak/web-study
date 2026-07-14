# 05. Next.js 연동 패턴 — server / client / middleware

> **핵심 질문**
> - 왜 Supabase 클라이언트를 서버용/브라우저용으로 따로 만들어야 하는가?
> - 서버 컴포넌트, 클라이언트 컴포넌트, Server Action, Route Handler, 미들웨어 — 각각 어떤 클라이언트를 쓰는가?
> - 세션 갱신(refresh)은 어디서 일어나야 하는가? (미들웨어의 역할)

## 개념 정리

### 왜 클라이언트가 여러 개인가 — 문제는 쿠키

Supabase 클라이언트 자체는 하나의 라이브러리다. 갈라지는 이유는 딱 하나: **세션이 쿠키에 있는데(02장), 쿠키를 읽고 쓰는 방법이 실행 컨텍스트마다 다르기 때문**이다.

| 컨텍스트 | 쿠키 읽기 | 쿠키 쓰기 |
|---|---|---|
| 브라우저 | `document.cookie` | `document.cookie` |
| 서버 컴포넌트 | `cookies()` (next/headers) | **불가** — 렌더링 중엔 응답 헤더를 못 만짐 |
| Server Action / Route Handler | `cookies()` | 가능 |
| 미들웨어 | `request.cookies` | `response.cookies` |

`@supabase/ssr`은 "쿠키를 이렇게 읽고 써라"를 주입받는 팩토리를 제공하고, 우리는 컨텍스트별로 그 주입이 다른 클라이언트 3종을 만든다.

### 관례 폴더 구조 — 3종 클라이언트

```
lib/supabase/
├── client.ts      # createBrowserClient — 클라이언트 컴포넌트용
├── server.ts      # createServerClient — 서버 컴포넌트/Server Action/Route Handler용
└── middleware.ts   # updateSession — 미들웨어용 세션 갱신
```

```ts
// lib/supabase/client.ts — 브라우저: 쿠키 접근이 자명해서 설정이 없다
import { createBrowserClient } from '@supabase/ssr'
export const createClient = () =>
  createBrowserClient(process.env.NEXT_PUBLIC_SUPABASE_URL!, process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!)
```

```ts
// lib/supabase/server.ts — 서버: Next의 cookies()를 주입
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!, process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => cookieStore.getAll(),
        setAll: (toSet) => {
          try { toSet.forEach(({ name, value, options }) => cookieStore.set(name, value, options)) }
          catch {} // 서버 컴포넌트에서 호출되면 쓰기 불가 — 미들웨어가 갱신을 담당하므로 무시해도 안전
        },
      },
    }
  )
}
```

**요청마다 새로 만든다**는 것에 주의 — 서버에서 클라이언트를 모듈 전역에 싱글턴으로 두면 A 유저의 요청이 B 유저의 세션을 볼 수 있다. `createClient()`를 매 요청 호출하는 것이 관례인 이유.

### 세션 갱신은 미들웨어에서 — `updateSession`

access token은 1시간이면 만료된다(02장). 누군가는 만료 전에 refresh token으로 갱신해야 하는데:

- 서버 컴포넌트는 쿠키를 **못 쓴다** → 갱신된 토큰을 저장할 수 없다.
- 클라이언트 컴포넌트만 믿으면 서버 렌더링 시점엔 이미 만료된 토큰으로 조회하게 된다.

그래서 **모든 요청보다 먼저 실행되고 쿠키를 쓸 수 있는 유일한 곳 = 미들웨어**가 세션 갱신 담당이 된다. 01-nextjs 04장에서 "미들웨어는 낙관적 확인 + 세션 갱신까지"라고 한 것의 후반부가 이것이다.

```ts
// lib/supabase/middleware.ts (핵심만)
export async function updateSession(request: NextRequest) {
  let response = NextResponse.next({ request })
  const supabase = createServerClient(URL, KEY, {
    cookies: {
      getAll: () => request.cookies.getAll(),
      setAll: (toSet) => {           // 갱신된 토큰을 요청·응답 양쪽 쿠키에 반영
        toSet.forEach(({ name, value }) => request.cookies.set(name, value))
        response = NextResponse.next({ request })
        toSet.forEach(({ name, value, options }) => response.cookies.set(name, value, options))
      },
    },
  })
  await supabase.auth.getUser()      // 이 호출이 만료 임박 토큰을 갱신시킨다
  return response
}
```

`middleware.ts`(루트)에서 `return await updateSession(request)` 하고, 로그인 여부에 따른 리다이렉트도 이 안에서 함께 처리한다(02장 실습에서 만든 것과 병합).

### 어디서 무엇을 — 데이터 페칭 위치 결정

| 상황 | 위치 | 클라이언트 |
|---|---|---|
| 초기 데이터 조회 | 서버 컴포넌트에서 조회 → props로 전달 | server |
| 변경(mutation) | Server Action + `revalidatePath` | server |
| 실시간 구독, 타이핑 중 검색 등 인터랙티브 | 클라이언트 컴포넌트 | browser |
| 외부에 노출할 API, OAuth 콜백 | Route Handler | server |

01-nextjs 03장의 "조회는 서버, 변경은 Server Action" 원칙이 그대로다 — Supabase가 끼어도 데이터 흐름의 모양은 안 바뀌고, `fetch` 자리가 `supabase.from()`으로 바뀔 뿐이다.

```ts
// 조회: app/memos/page.tsx (서버 컴포넌트)
const supabase = await createClient()
const { data: memos } = await supabase.from('memos').select().order('created_at', { ascending: false })
// where user_id = ... 를 안 썼다 — RLS(03장)가 이미 내 것만 돌려준다

// 변경: Server Action
'use server'
export async function createMemo(formData: FormData) {
  const supabase = await createClient()
  await supabase.from('memos').insert({ title: formData.get('title') as string, user_id: user.id })
  revalidatePath('/memos')
}
```

### 타입 안전성 — `gen types`

```bash
supabase gen types typescript --local > lib/database.types.ts
```

생성된 `Database` 타입을 `createClient<Database>()`에 물리면 `.from('memos')`의 테이블명, `.select()` 결과, `.insert()` 페이로드가 전부 스키마 기준으로 타입 검사된다. **DB 스키마가 곧 타입의 소스**가 되는 것 — 스키마를 바꾸면(마이그레이션) 타입을 재생성하고, 어긋난 코드는 컴파일이 잡는다. 01-nextjs 09장에서 zod로 "런타임 입력"을 검증했다면, 이건 "DB와 코드 사이"를 컴파일 타임에 검증하는 것으로 역할이 다르다.

## 실습 (01-nextjs practice 앱에)

- [ ] `lib/supabase/` 3종 클라이언트 + `middleware.ts`의 `updateSession`을 관례 구조대로 셋업
  → **의미**: 모든 Supabase × Next.js 프로젝트의 뼈대를 한 번 손으로 만든다. 이후 새 프로젝트(05-capstone 포함)에서는 이 구조를 그대로 복사하는 출발점이 된다.
- [ ] 메모 CRUD를 in-memory 배열 → Supabase로 전면 교체: 조회는 서버 컴포넌트, 변경은 Server Action + `revalidatePath`
  → **의미**: 01-nextjs에서 만든 앱이 처음으로 진짜 영속성을 갖는다(서버 재시작해도 데이터 유지). "백엔드 코드 없이 DB까지 갔다"는 것이 이 챕터 전체의 결론이고, 그 한계를 03-nestjs에서 이어받는다.
- [ ] 쿼리에서 `where user_id` 필터를 일부러 빼고도 내 메모만 나오는 것 확인
  → **의미**: 앱 코드의 필터가 아니라 RLS가 데이터를 지키고 있음을 재확인한다. "필터를 깜빡해도 유출이 없다"가 이 아키텍처의 안전 마진이다.
- [ ] `gen types`로 타입 생성 후 스키마에 컬럼 하나 추가 → 타입 재생성 전/후 컴파일 에러 변화 관찰
  → **의미**: "스키마 = 타입 소스" 루프(마이그레이션 → 타입 재생성 → 컴파일 검증)를 한 바퀴 돌려본다. 이 루프가 습관이 되면 DB와 코드가 어긋난 채 배포되는 사고가 구조적으로 막힌다.

## 참고 자료

- https://supabase.com/docs/guides/auth/server-side/nextjs — 이 노트 패턴의 원전 (코드 그대로 제공)
- https://supabase.com/docs/guides/api/rest/generating-types
