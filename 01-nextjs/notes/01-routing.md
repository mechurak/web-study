# 01. 라우팅 — App Router

> **핵심 질문**
> - 파일시스템 기반 라우팅이란? `app/` 폴더 구조가 URL과 어떻게 대응되는가?
> - `layout.tsx`는 `page.tsx`와 무엇이 다르고, 왜 리렌더링되지 않는가?
> - Pages Router와 App Router의 차이는? 왜 App Router로 넘어갔는가?

## 개념 정리

### 파일시스템 기반 라우팅

라우팅 테이블을 코드로 등록하는 대신(React Router의 `<Route path=...>`), **폴더 구조가 곧 URL**이다:

```
app/
├── page.tsx              →  /
├── memos/
│   ├── page.tsx          →  /memos
│   └── [id]/
│       └── page.tsx      →  /memos/123
```

규칙: **폴더 = URL 세그먼트, `page.tsx` = 그 URL의 화면**. `page.tsx`가 없는 폴더는 URL이 생기지 않는다(구조화용). 이 설계의 의도는 "라우트가 어디 정의됐지?"를 찾아다닐 필요 없이 폴더만 보면 서비스의 전체 URL 지도가 보이게 하는 것.

### 특수 파일들 — 폴더마다 깔 수 있는 UI 계층

| 파일 | 역할 | 언제 보이나 |
|---|---|---|
| `page.tsx` | 그 경로의 화면 (필수) | URL 접속 시 |
| `layout.tsx` | 하위 경로들이 공유하는 껍데기 | 항상 (page를 감쌈) |
| `loading.tsx` | 로딩 UI | page가 데이터 기다리는 동안 (자동 Suspense) |
| `error.tsx` | 에러 UI | 하위에서 에러 throw 시 (자동 Error Boundary) |
| `not-found.tsx` | 404 UI | `notFound()` 호출 시 |

**`layout.tsx`가 `page.tsx`와 결정적으로 다른 점**: 페이지를 이동해도 레이아웃은 **리렌더링되지 않고 상태가 유지된다**. `/memos/1` → `/memos/2`로 이동하면 `[id]/page.tsx`만 교체되고 상위 레이아웃(네비게이션 바, 사이드바)은 그대로다. 그래서 레이아웃에 둔 스크롤 위치·입력값·재생 중인 영상이 페이지 이동에도 살아남는다. 공통 UI를 매 페이지에서 import하는 게 아니라 "위치"로 선언하는 설계.

### 중첩 레이아웃

레이아웃은 폴더 계층을 따라 중첩된다. `app/layout.tsx`(루트, `<html><body>` 필수) 안에 `app/memos/layout.tsx`가 들어가고, 그 안에 page가 들어간다. 각 구역(대시보드, 마케팅 페이지)마다 다른 껍데기를 계층적으로 조립할 수 있다.

### 동적 라우트

| 문법 | 매칭 | 예 |
|---|---|---|
| `[id]` | 세그먼트 1개 | `/memos/1` (o), `/memos/1/2` (x) |
| `[...slug]` | 1개 이상 전부 | `/docs/a/b/c` → `slug = ['a','b','c']` |
| `[[...slug]]` | 0개 이상 전부 | `/docs`도 매칭 (slug = undefined) |

**Next 15부터 `params`는 Promise다** — await 해야 한다 (14까지는 동기 객체였음):

```tsx
// app/memos/[id]/page.tsx
export default async function MemoPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  // id로 데이터 조회...
}
```

`searchParams`(쿼리스트링)도 마찬가지로 Promise. 이렇게 바뀐 이유: 요청에 의존하는 값을 비동기로 만들어, 그 값을 실제로 쓰기 전까지는 정적으로 미리 렌더링할 수 있게 하려는 설계다.

### 네비게이션

- **`<Link href="/memos">`**: `<a>`를 감싸지만 동작이 다르다 — 문서 전체를 다시 받지 않고 바뀐 세그먼트만 가져와 교체(클라이언트 사이드 전환)하고, 뷰포트에 보이면 대상 페이지를 **prefetch**해둔다. 그래서 클릭 순간 거의 즉시 뜬다. 생 `<a>`를 쓰면 풀 리로드라 레이아웃 상태 유지가 깨진다.
- **`useRouter()`** (클라이언트 컴포넌트): `router.push/replace/back` — 이벤트 핸들러에서 프로그래밍 방식 이동.
- **`redirect('/login')`** (서버): 서버 컴포넌트/Server Action에서 리다이렉트. return이 필요 없다(throw로 동작).
- **`usePathname()` / `useSearchParams()`**: 현재 경로/쿼리 읽기 — 네비게이션 바의 활성 메뉴 표시에 사용.

### 고급 — 나중에 필요할 때 다시 볼 것

- **Route Groups `(marketing)`**: 괄호 폴더는 URL에 안 나타난다. `/about`을 유지하면서 마케팅 페이지들만 다른 레이아웃으로 묶고 싶을 때 사용.
- **Parallel Routes `@modal` / Intercepting Routes `(.)`**: 한 화면에 독립적인 슬롯 여러 개(대시보드 위젯), 또는 "목록에서 클릭하면 모달, URL 직접 접근하면 전체 페이지"(인스타그램 사진 뷰어 패턴). 문법이 복잡하니 필요해질 때 공식 문서로.

### Pages Router와의 차이 (구버전 코드를 읽을 때)

`pages/` 폴더 기반의 구세대 라우터. 파일 하나 = 페이지 하나이고 레이아웃 중첩·서버 컴포넌트·streaming이 없다. App Router로 넘어간 핵심 이유가 **서버 컴포넌트(02장)와 레이아웃 계층** 지원이다. 검색하다 `getServerSideProps`, `_app.tsx`가 나오면 Pages Router 자료이니 걸러 읽을 것.

## 실습 (practice 앱에서)

- [ ] 홈(`/`) / 메모 목록(`/memos`) / 메모 상세(`/memos/[id]`) 3단 구조 페이지 만들기
  → **의미**: 폴더 = URL 대응과 Next 15의 `await params`를 손으로 확인한다. 이 3단 구조가 이후 모든 챕터 실습(데이터 페칭, Supabase 연동)의 뼈대가 된다.
- [ ] 공통 네비게이션 바를 루트 레이아웃에 배치하고, 페이지를 이동해도 상태(예: 네비게이션 안의 input 입력값)가 유지되는 것 확인
  → **의미**: "레이아웃은 리렌더링되지 않는다"를 눈으로 검증. `<a>`로 바꿔서 풀 리로드와 비교해보면 `<Link>`의 가치가 체감된다.
- [ ] `loading.tsx` 추가 후 페이지에 인위적 딜레이(`await new Promise(r => setTimeout(r, 2000))`)를 넣어 로딩 UI 확인, 존재하지 않는 id에는 `notFound()` 호출
  → **의미**: 특수 파일들이 자동으로 Suspense/에러 경계가 되는 구조를 확인. 02장(streaming)과 03장(느린 데이터 페칭)에서 이 로딩 UI가 계속 등장한다.
- [ ] `usePathname()`으로 네비게이션 바의 현재 메뉴 하이라이트
  → **의미**: 클라이언트 훅을 쓰려면 `"use client"`가 필요하다는 첫 경험 — 02장 서버/클라이언트 컴포넌트 경계 논의의 예고편.

## 참고 자료

- https://nextjs.org/docs/app/building-your-application/routing
- https://nextjs.org/docs/app/api-reference/file-conventions (특수 파일 전체 목록)
