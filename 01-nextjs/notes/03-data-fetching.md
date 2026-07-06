# 03. 데이터 페칭 — fetch 캐싱, Server Actions

> **핵심 질문**
> - Next.js의 `fetch`는 브라우저 fetch와 뭐가 다른가? (캐싱, 중복 제거)
> - Server Action이란? API Route 없이 어떻게 서버 코드를 호출하는가?
> - 언제 Server Action을 쓰고, 언제 별도 API(Route Handler / NestJS)를 쓰는가?

## 개념 정리

### 서버 컴포넌트에서의 데이터 페칭

App Router의 기본 형태: 컴포넌트가 `async` 함수가 되고, 그 안에서 바로 `await`한다. useEffect + useState + 로딩 플래그의 삼중주가 사라진다.

```tsx
// app/memos/page.tsx — 서버 컴포넌트
export default async function MemosPage() {
  const res = await fetch('https://api.example.com/memos')
  const memos = await res.json()
  return <ul>{memos.map(m => <li key={m.id}>{m.title}</li>)}</ul>
}
```

이게 가능한 이유는 이 코드가 **서버에서만 실행**되기 때문(02장). API 키를 헤더에 넣어도 브라우저에 노출되지 않고, DB라면 fetch 없이 직접 쿼리해도 된다(02-supabase 05장에서 그렇게 한다).

### fetch 캐싱 — Next가 확장한 부분

Next.js는 서버에서의 `fetch`에 캐시 계층을 끼워 넣었다(Data Cache — 요청이 끝나도 남는 서버 측 캐시).

```ts
fetch(url)                                 // 기본: 캐시 안 함 (매번 요청)
fetch(url, { cache: 'force-cache' })       // 캐시함 — 같은 URL은 다시 안 감
fetch(url, { next: { revalidate: 60 } })   // 캐시하되 60초 후 재검증 (ISR과 같은 원리)
fetch(url, { next: { tags: ['memos'] } })  // 태그를 달아 나중에 골라서 무효화
```

> **버전 주의**: Next 14까지는 `fetch`가 **기본 캐시**였다(`force-cache`가 기본값). **Next 15부터 기본이 캐시 안 함**(`no-store`)으로 뒤집혔다 — "왜 데이터가 안 바뀌지?"라는 혼란이 너무 많았기 때문. 구버전 자료를 읽을 때 이 차이를 반드시 염두에 둘 것.

**요청 중복 제거(deduplication)는 캐싱과 별개로 항상 동작한다**: 한 번의 렌더링 중 여러 컴포넌트가 같은 `fetch(url)`을 호출하면 실제 요청은 1번만 나간다. 그래서 "데이터를 최상위에서 받아 props로 내려보내야 한다"는 강박 없이, **필요한 컴포넌트가 각자 fetch**해도 된다 — 컴포넌트의 독립성을 지키는 설계. fetch가 아닌 함수(DB 쿼리 등)는 React의 `cache()`로 감싸면 같은 효과를 얻는다.

### 캐시 무효화 — 두 가지 축

| 방식 | 트리거 | 예 |
|---|---|---|
| 시간 기반 | `revalidate: 60` — 60초 지나면 다음 요청 때 재생성 | 뉴스 목록, 상품 페이지 |
| 온디맨드 | 코드에서 `revalidatePath('/memos')` / `revalidateTag('memos')` 호출 | 글 작성 직후 목록 갱신 |

온디맨드가 더 정확하다 — "데이터가 바뀐 순간"을 아는 것은 변경을 처리한 코드 자신이기 때문. 그래서 Server Action(아래)과 짝으로 쓰인다: 변경 → `revalidatePath` → 다음 렌더링은 신선한 데이터.

### Server Actions — 함수 호출처럼 보이는 서버 엔드포인트

`"use server"`를 선언한 함수는 클라이언트에서 import해서 "호출"할 수 있다. 실제로는 Next가 그 함수마다 숨은 POST 엔드포인트를 만들고, 호출을 네트워크 요청으로 변환한다. **API Route를 직접 만들고 fetch로 부르는 왕복 코드가 사라지는 것**이 가치다.

```ts
// app/memos/actions.ts
'use server'
export async function createMemo(formData: FormData) {
  const title = formData.get('title') as string
  if (!title) return { error: '제목은 필수' }     // 검증 — 09장에서 zod로 강화
  await db.insert({ title })
  revalidatePath('/memos')                        // 변경 후 캐시 무효화
}
```

```tsx
// 폼에 직접 연결 — JS 로드 전에 제출해도 동작한다 (progressive enhancement)
<form action={createMemo}> ... </form>
```

폼의 진행 상태는 React 훅으로 다룬다:
- `useActionState(action, initialState)` — 액션의 반환값(에러 메시지 등)과 pending 상태를 함께 관리. (React 19에서 `useFormState`를 대체한 이름)
- `useFormStatus()` — `<form>` 안쪽 컴포넌트에서 제출 중 여부를 읽어 버튼 비활성화 등에 사용.

**보안 유의점 — 이 챕터에서 가장 중요한 문장**: Server Action은 함수처럼 보이지만 **결국 공개 HTTP 엔드포인트다**. 화면에서 폼으로만 호출된다고 해서 악의적 직접 호출이 불가능한 게 아니다. 따라서 모든 액션 안에서 (1) 입력 검증(zod, 09장), (2) 인증 확인(로그인했는가), (3) 인가 확인(이 리소스의 주인인가)을 해야 한다. "UI에서 버튼을 숨겼으니 안전"은 보안이 아니다(04-deployment 06장).

### 언제 Server Action, 언제 별도 API?

| 상황 | 선택 |
|---|---|
| 내 Next 앱의 폼 제출, 간단한 변경 | Server Action |
| 외부(모바일 앱, 타 서비스)도 호출해야 하는 API | Route Handler (04장) 또는 NestJS |
| 복잡한 비즈니스 로직, 트랜잭션, 장시간 작업 | NestJS (03-nestjs) |

Server Action은 "이 Next 앱 전용 내부 통로"다. 공개 API 계약이 필요해지는 순간이 별도 백엔드로 넘어가는 신호 중 하나.

### 클라이언트 페칭은 사라지지 않았다

서버 컴포넌트가 **초기 데이터**를 담당해도, 마운트 이후 계속 살아 움직이는 데이터 — 폴링, 무한스크롤, 낙관적 업데이트, 포커스 시 재검증 — 는 클라이언트의 영역이다. 이때 생 useEffect 대신 TanStack Query를 쓴다. → [08-state-management.md](./08-state-management.md)에서 본격적으로.

## 실습 (practice 앱에서)

- [ ] 외부 공개 API(JSONPlaceholder 등)를 서버 컴포넌트에서 fetch해 목록 렌더링. `cache: 'force-cache'` / `revalidate: 10` / 기본값 세 가지로 바꿔가며 새로고침 시 동작 차이 확인
  → **의미**: Next 15의 캐시 기본값과 옵션별 차이를 직접 확인한다. 02장의 SSG/ISR/SSR 구분이 사실 fetch 캐시 옵션과 같은 축임을 발견하는 것이 목표.
- [ ] 같은 fetch를 페이지와 그 하위 컴포넌트 두 곳에서 호출하고, API 서버 로그(또는 콘솔 카운터)로 실제 요청이 1번인 것 확인
  → **의미**: 중복 제거를 믿을 수 있게 되면 "각 컴포넌트가 자기 데이터를 알아서 가져오는" 구조로 설계할 수 있다 — props drilling 없는 서버 컴포넌트 트리의 근거.
- [ ] 메모 추가 폼을 Server Action으로 구현 (저장은 일단 메모리 배열, 02-supabase에서 DB로 교체) + `revalidatePath`로 목록 갱신 + `useActionState`로 검증 에러 표시
  → **의미**: "변경 → 무효화 → 재렌더링" 사이클이 이 스택의 표준 쓰기 경로다. 여기서 만든 폼이 07장(shadcn Form), 09장(zod)에서 계속 업그레이드된다.
- [ ] JS를 끈 브라우저(DevTools에서 비활성화)에서 그 폼을 제출해보기
  → **의미**: progressive enhancement가 실제로 동작하는 것 확인 — Server Action이 "fetch를 감춘 문법 설탕"이 아니라 HTML 폼 위에 쌓인 설계임을 이해한다.

## 참고 자료

- https://nextjs.org/docs/app/building-your-application/data-fetching
- https://nextjs.org/docs/app/building-your-application/caching (캐시 4계층 전체 그림 — 어려우니 나중에 다시)
- https://react.dev/reference/rsc/server-functions
