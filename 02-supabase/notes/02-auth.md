# 02. 인증 — 이메일/소셜 로그인, 세션

> **핵심 질문**
> - 인증(Authentication)과 인가(Authorization)의 차이는?
> - JWT(JSON Web Token) 기반 세션은 어떻게 동작하는가? access token / refresh token의 역할은?
> - 서버 컴포넌트에서 "현재 로그인한 유저"를 어떻게 아는가? (쿠키 기반 세션)

## 개념 정리

### 인증 vs 인가 — 이 챕터의 지도

- **인증(Authentication)** = "너는 누구인가"를 확인하는 것. 로그인. → 이 노트(02장)의 주제.
- **인가(Authorization)** = "그 너가 이걸 해도 되는가"를 판단하는 것. → Supabase에서는 RLS(03장)의 주제.

Supabase Auth는 인증을 맡아 "이 요청은 유저 X의 것"이라는 사실을 토큰으로 만들어주고, RLS가 그 사실을 받아 인가를 수행한다. 두 챕터가 한 세트다.

### JWT 세션의 동작 원리

로그인에 성공하면 Supabase Auth는 토큰 두 개를 준다:

| | access token (JWT) | refresh token |
|---|---|---|
| 정체 | "유저 X임"이 서명된 증명서 | 새 access token을 받아오는 교환권 |
| 수명 | 짧다 (기본 1시간) | 길다 (재사용 시 회전) |
| 검증 방식 | **서명만 확인** — DB 조회 불필요 | Auth 서버가 DB에서 확인 |

**JWT의 구조**는 `header.payload.signature` 세 부분이다. payload에 `sub`(유저 id), `role`, 만료 시각(`exp`)이 들어 있고, signature는 Supabase만 아는 비밀키로 서명돼 있다. 누구나 payload를 **읽을 수는** 있지만(민감정보 넣지 말 것) **위조는** 불가 — 내용을 한 글자라도 바꾸면 서명이 안 맞는다.

**왜 이렇게 설계됐는가**: 매 요청마다 DB에서 세션을 조회하면 그 자체가 병목이다. JWT는 서명 검증만으로 "유저 X"를 신뢰할 수 있어서, PostgREST가 DB 조회 없이 모든 요청에 유저를 태울 수 있다. RLS 정책 안의 `auth.uid()`가 바로 이 JWT payload의 `sub`를 읽는 것이다(→ 03장). 대신 JWT는 발급 후 회수가 어렵다(서버가 상태를 안 가지니까) — 그래서 수명을 짧게 하고 refresh token으로 갱신하는 구조가 된다.

### 로그인 방법들

- **이메일 + 비밀번호** — `signUp()` / `signInWithPassword()`. 기본값으로 확인 메일을 보낸다(로컬에선 Inbucket에 잡힘 → 00장 §5).
- **매직 링크 / OTP(One-Time Password)** — 비밀번호 없이 이메일로 일회성 링크/코드 발송. 비밀번호 유출·재설정 흐름이 통째로 사라지는 대신 메일 의존이 생긴다.
- **소셜 로그인(OAuth)** — `signInWithOAuth({ provider: 'github' })`. 핵심은 리다이렉트 체인이다:

```
내 앱 → GitHub 로그인 → GitHub가 Supabase로 리다이렉트(redirect URI)
     → Supabase가 code를 들고 내 앱 /auth/callback으로 → code를 세션으로 교환
```

로그인의 상대방이 내 앱이 아니라 **Supabase Auth 서버**라는 것이 모든 설정값의 이유다 — GitHub OAuth 앱의 redirect URI에 `supabase.co` 주소를 넣는 이유, Supabase 대시보드에 Redirect URLs 허용 목록이 있는 이유(open redirect 방지) 모두 여기서 나온다. 구체 절차는 [00-setup.md §7](./00-setup.md)(구글 기준, GitHub도 동일 구조).

`/auth/callback`은 Route Handler로 만든다:

```ts
// app/auth/callback/route.ts
import { NextResponse } from 'next/server'
import { createClient } from '@/lib/supabase/server'

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url)
  const code = searchParams.get('code')
  if (code) {
    const supabase = await createClient()
    await supabase.auth.exchangeCodeForSession(code)   // code → 세션(쿠키에 저장)
  }
  return NextResponse.redirect(`${origin}/memos`)
}
```

### 세션은 어디에 저장되는가 — localStorage vs cookie

`supabase-js`의 기본 동작은 세션을 **localStorage**에 저장한다. 브라우저만 있는 SPA(Single Page Application)라면 충분하다. 그런데 Next.js에는 서버가 있다:

- localStorage는 **브라우저 전용**이다. 서버 컴포넌트가 렌더링될 때 서버는 localStorage를 볼 수 없다 → "현재 로그인한 유저"를 알 수 없다.
- **쿠키**는 매 HTTP 요청에 자동으로 실려 서버에 도착한다 → 서버 컴포넌트/Server Action/미들웨어가 전부 세션을 읽을 수 있다.

그래서 Next.js에서는 `@supabase/ssr` 패키지로 세션 저장소를 쿠키로 바꾼다. 이것이 "핵심 질문 3"의 답이다: **서버 컴포넌트는 요청에 실려온 세션 쿠키를 읽어 현재 유저를 안다.** 쿠키를 읽고 쓰는 위치가 컨텍스트(브라우저/서버/미들웨어)마다 달라서 클라이언트를 3종으로 나눠 만들게 되는데, 그 구체 패턴은 05장에서 다룬다.

### 보안 기본기

- **`anon key` vs `service_role key`** — anon key는 공개돼도 되는 식별자(권한은 RLS가 통제), service_role key는 RLS를 우회하는 관리자 키로 절대 클라이언트에 노출 금지. 상세는 [00-setup.md §3](./00-setup.md).
- **`getUser()` vs `getSession()`** — 서버 코드에서 신뢰가 필요한 판단(권한 분기 등)에는 `getUser()`를 쓴다. `getSession()`은 쿠키의 값을 검증 없이 돌려줄 수 있지만, `getUser()`는 Auth 서버에 토큰을 검증시킨다. "쿠키는 클라이언트가 보낸 입력"이라는 원칙의 적용.
- **운영 설정** — 이메일 확인(Confirm email) 켜기, 비밀번호 최소 길이, rate limit(무차별 대입 방지)은 대시보드 Authentication 설정에 있다. 기본값이 무난하지만 "있다는 것"은 알아둘 것.

## 실습 (01-nextjs practice 앱에)

- [ ] 이메일 회원가입/로그인/로그아웃 구현 (`signUp`, `signInWithPassword`, `signOut` + 로컬 Inbucket에서 확인 메일 열기)
  → **의미**: 세션의 생성과 소멸을 직접 다룬다. 로그인 후 쿠키(개발자도구 → Application → Cookies)에 뭐가 생기는지 눈으로 확인하면 "세션 = 쿠키에 실린 토큰"이 추상이 아니게 된다.
- [ ] GitHub OAuth 로그인 추가 (OAuth 앱 등록 → provider 설정 → `/auth/callback` 핸들러)
  → **의미**: 리다이렉트 체인(앱→GitHub→Supabase→앱)을 한 번 완주한다. 설정이 두 군데(GitHub, Supabase 대시보드)로 갈리는 이유를 이해하면 다른 어떤 provider도 같은 요령으로 붙일 수 있다.
- [ ] 로그인 상태에 따라 네비게이션 바 UI 분기 (서버 컴포넌트에서 `getUser()`로 조회)
  → **의미**: "서버가 쿠키로 현재 유저를 안다"의 실물. 01-nextjs 02장의 서버 컴포넌트 지식과 인증이 만나는 지점이다.
- [ ] 미들웨어에서 미로그인 시 보호 페이지 접근 차단 — 01-nextjs 04장에서 "아무 쿠키나"로 만들었던 미들웨어를 진짜 세션 검사로 교체
  → **의미**: 04장에서 잡아둔 "전역 관문 + 낙관적 확인" 자리에 진짜 인증을 끼운다. 미들웨어의 또 다른 역할(세션 갱신)은 05장 `updateSession`에서 완성된다.

## 참고 자료

- https://supabase.com/docs/guides/auth
- https://supabase.com/docs/guides/auth/server-side/nextjs — 쿠키 기반 세션 공식 가이드
- https://jwt.io — JWT를 붙여넣으면 payload를 해부해서 보여준다
