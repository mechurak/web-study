# 04. API Routes — Route Handlers, 미들웨어

> **핵심 질문**
> - Next.js 안에서 백엔드 API를 만들 수 있는데, 왜 NestJS 같은 별도 백엔드가 필요해지는가?
> - Route Handler는 어떤 런타임에서 실행되는가? (Node vs Edge)
> - 미들웨어로 할 수 있는 것과 하면 안 되는 것은?

## 개념 정리

### Route Handlers — Next 안의 API 엔드포인트

`app/api/.../route.ts` 파일에 HTTP 메서드 이름의 함수를 export하면 그 경로가 API가 된다. 화면(`page.tsx`)과 같은 라우팅 규칙을 쓰되, HTML 대신 JSON을 반환하는 것.

```ts
// app/api/memos/route.ts  →  /api/memos
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const query = request.nextUrl.searchParams.get('q')   // 쿼리스트링
  const memos = await findMemos(query)
  return NextResponse.json(memos)
}

export async function POST(request: NextRequest) {
  const body = await request.json()                     // 바디 파싱
  if (!body.title) {
    return NextResponse.json({ error: 'title 필수' }, { status: 400 })
  }
  const memo = await createMemo(body)
  return NextResponse.json(memo, { status: 201 })
}
```

- `NextRequest`/`NextResponse`는 웹 표준 `Request`/`Response`의 확장이다(쿠키·URL 헬퍼 추가). 표준 `Response.json()`을 반환해도 된다 — Next만의 독자 규격이 아니라는 점이 설계 의도.
- 동적 세그먼트도 페이지와 동일: `app/api/memos/[id]/route.ts`에서 두 번째 인자로 `{ params }`를 받는다 (Next 15부터 **params는 Promise** — `const { id } = await params`).
- 같은 경로에 `page.tsx`와 `route.ts`를 둘 다 둘 수는 없다 — 한 URL의 응답 주인은 하나.
- 캐싱: Route Handler의 GET은 **기본적으로 동적**(매 요청 실행)이다. 14 시절 "GET은 기본 캐시" 동작이 15에서 뒤집혔다(03장 fetch 기본값 변화와 같은 방향).

**Server Action(03장)과의 관계**: 내 앱의 폼 제출이면 Server Action이 간편하고, Route Handler는 **URL로 노출되는 공개 계약**이 필요할 때 쓴다 — 모바일 앱이 호출할 API, 외부 서비스가 부르는 webhook 수신(결제 알림 등), RSS/사이트맵 같은 비HTML 응답.

### 미들웨어 — 모든 요청 앞의 검문소

프로젝트 루트의 `middleware.ts` 하나가 매칭되는 **모든 요청보다 먼저** 실행된다. 라우트별 코드가 아니라 전역 관문이다.

```ts
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  const session = request.cookies.get('session')
  if (!session && request.nextUrl.pathname.startsWith('/memos')) {
    return NextResponse.redirect(new URL('/login', request.url))   // 차단
  }
  return NextResponse.next()                                        // 통과
}

export const config = {
  matcher: ['/memos/:path*', '/settings/:path*'],   // 적용 경로 제한
}
```

- **적합한 일**: 로그인 여부에 따른 리다이렉트, 헤더 추가/수정, A/B 테스트 분기, 지역별 라우팅. 공통점은 "가볍고 빠른 판단".
- **하면 안 되는 일**: DB 조회, 무거운 연산. 미들웨어는 **모든 매칭 요청**에 실행되므로 여기서 느리면 사이트 전체가 느려진다. 또한 기본 런타임이 Edge라서 Node 전용 API(fs, net, 대부분의 DB 드라이버)를 쓸 수 없다.
- 그래서 인증에서 미들웨어의 역할은 "세션 쿠키가 **있는지**" 수준의 낙관적 확인 + 세션 갱신까지다. "이 유저가 이 리소스의 주인인지" 같은 진짜 검사는 데이터에 접근하는 곳(서버 컴포넌트, Server Action, 백엔드)에서 다시 한다. → 02-supabase 05장의 `updateSession` 패턴에서 실물을 만든다.

### 런타임 두 가지 — Node vs Edge

| | Node.js 런타임 (기본) | Edge 런타임 |
|---|---|---|
| 실행 위치 | 오리진 서버(리전 1곳) | 전 세계 엣지 (유저 가까이) |
| 시작 속도 | cold start 있음 | 매우 빠름 |
| 사용 가능 API | Node 전부 (fs, 모든 npm) | 웹 표준 API만 (fetch, Request/Response) |
| 쓰는 곳 | Route Handler, 서버 컴포넌트 | 미들웨어(기본), 선택적으로 핸들러 |

Edge는 "빠르지만 제한된 환경"이다. 미들웨어가 Edge인 이유: 모든 요청이 거치는 관문이니 유저 가까이에서 즉시 실행돼야 하기 때문. (최근 버전에서는 미들웨어에 Node 런타임을 선택하는 옵션도 생겼다 — 세부는 공식 문서 확인.) 04-deployment 01장에서 Vercel 인프라 관점으로 다시 본다.

### Next API의 한계 — NestJS가 필요해지는 지점

이 레포의 학습 서사에서 중요한 섹션이다. Route Handler로 API를 만들다 보면 다음 균열들을 만나게 되고, 그것이 03-nestjs로 넘어가는 이유가 된다:

1. **구조가 없다.** Route Handler는 "URL당 함수 하나"가 전부다. 검증, 권한 검사, 비즈니스 로직, DB 접근이 전부 한 함수에 쌓인다. 엔드포인트가 30개가 되면 — 공통 로직은 어디에? 트랜잭션 경계는? 팀원마다 다르게 짠다. NestJS는 모듈/DI/레이어(컨트롤러-서비스-리포지토리)로 이 답을 **프레임워크가 강제**한다.

2. **serverless와 안 맞는 작업들.** Vercel의 함수는 요청-응답 후 사라지고 실행 시간 제한이 있다. 그래서 상시 프로세스가 필요한 것들이 안 된다:
   - 장시간 작업 (동영상 인코딩, 대량 메일 발송)
   - WebSocket 상시 연결 (실시간 협업)
   - 크론/배치, 작업 큐 (매일 자정 정산)
   NestJS는 상시 서버라 이 전부가 자연스럽다.

3. **프론트와 백엔드의 생명주기가 묶인다.** API만 고쳐도 프론트와 함께 배포되고, 스케일링도 함께 간다. 모바일 앱 등 **다른 클라이언트가 생기는 순간** API는 독립된 제품이 돼야 한다 — 별도 버전 관리, 별도 배포, 명세(Swagger) 제공.

거꾸로 말하면: 이 균열들을 만나기 전까지는 **Next + Supabase로 충분하다**. "백엔드는 원래 만드는 것"이 아니라, 균열이 보일 때 도입하는 것 — 그 감각을 익히는 것이 01→02→03 순서의 목적이다.

## 실습 (practice 앱에서)

- [ ] 메모 CRUD를 Route Handler로 구현: `GET/POST /api/memos`, `GET/DELETE /api/memos/[id]` (저장은 03장과 같은 메모리 배열 공유). curl 또는 브라우저로 직접 호출해 상태 코드(200/201/400/404)까지 확인
  → **의미**: "화면 없이 URL로만 존재하는 응답"을 만들어본다. 여기서 짠 CRUD가 03-nestjs 02장에서 컨트롤러/서비스/DTO로 재구현되는 비교 대상이 된다 — 같은 기능을 두 방식으로 짜봐야 구조의 가치가 보인다.
- [ ] POST 핸들러에 검증·에러 응답을 점점 추가해보며 함수가 비대해지는 것을 관찰하고, "이걸 30개 엔드포인트에서 반복한다면?"을 상상해 느낀 점을 이 노트 하단에 기록
  → **의미**: NestJS 도입 동기를 몸으로 만드는 실습. 03-nestjs 진입 시 이 기록을 다시 읽으면 DI/파이프/가드가 "무엇의 해결책"인지 명확해진다.
- [ ] `middleware.ts` 작성: 쿠키가 없으면 `/memos`를 `/login`으로 리다이렉트 (지금은 아무 쿠키나, 진짜 세션은 02-supabase 02장에서 교체). matcher로 적용 범위 제한
  → **의미**: "전역 관문 + 낙관적 확인"이라는 미들웨어의 위치를 잡는다. 02-supabase의 세션 갱신 미들웨어가 이 파일 위에 얹힌다.
- [ ] 미들웨어 안에서 일부러 Node API(`fs.readFileSync`)를 호출해 에러 확인
  → **의미**: Edge 런타임 제약을 에러 메시지로 직접 만난다. "미들웨어에서 DB 조회하면 안 되는" 이유의 절반(나머지 절반은 성능)을 체감.

## 참고 자료

- https://nextjs.org/docs/app/building-your-application/routing/route-handlers
- https://nextjs.org/docs/app/building-your-application/routing/middleware
- https://nextjs.org/docs/app/api-reference/edge (Edge 런타임에서 되는 것/안 되는 것)
