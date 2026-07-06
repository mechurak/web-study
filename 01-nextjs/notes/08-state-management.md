# 08. 상태 관리 — 서버 상태 vs 클라이언트 상태

> **핵심 질문**
> - "서버 상태"와 "클라이언트 상태"는 무엇이 다른가? 왜 다른 도구로 관리하는가?
> - 어떤 데이터가 생겼을 때, 서버 컴포넌트 / TanStack Query / useState / Zustand 중 무엇으로 다룰지 판단하는 기준은?
> - Redux가 표준이던 시대에서 왜 이렇게 바뀌었는가? (서버 상태를 전역 스토어에 넣던 시절의 문제)

## 개념 정리

### 상태의 분류부터

"상태 관리 라이브러리 뭐 쓰지?"보다 먼저 물을 것: **이 데이터의 원본(source of truth)은 어디인가?**

| 종류 | 원본 | 예 | 특징 |
|---|---|---|---|
| **서버 상태** | 서버의 DB | 메모 목록, 유저 프로필 | 내가 안 바꿔도 변한다(다른 유저, 다른 탭). 브라우저가 들고 있는 건 **캐시일 뿐** |
| **클라이언트 상태** | 브라우저 메모리 | 모달 열림, 선택된 탭, 입력 중인 폼 | 나만 바꾼다. 새로고침하면 사라져도 됨 |
| **URL 상태** | 주소창 | 검색어, 필터, 페이지 번호 | 공유·북마크·새로고침에도 살아남아야 하는 상태 |

이 구분이 도구 선택을 결정한다. 서버 상태의 본질은 "언제 낡은가(stale), 언제 다시 가져오나"라는 **캐시 동기화 문제**이고, 클라이언트 상태는 그냥 변수다. Redux 시대의 고통은 이 둘을 한 스토어에 섞어서, 캐시 무효화·로딩/에러 상태·중복 요청 제거를 전부 손으로 짰던 것이다. TanStack Query가 그 캐시 문제를 통째로 가져가면서, 남는 클라이언트 상태는 대부분 `useState`로 충분할 만큼 작아졌다.

### 서버 상태 — TanStack Query

**기본 형태** — 서버 상태를 `queryKey`로 식별되는 캐시 엔트리로 다룬다:

```tsx
const { data, isPending, error } = useQuery({
  queryKey: ['memos', { filter }],   // 캐시의 주소. 의존하는 값은 전부 키에 넣는다
  queryFn: () => fetchMemos(filter),
})

const { mutate } = useMutation({
  mutationFn: deleteMemo,
  onSettled: () => queryClient.invalidateQueries({ queryKey: ['memos'] }),
})
```

**staleTime / gcTime — "신선함"의 두 타이머** (v5 기준, `gcTime`은 구 `cacheTime`):

- `staleTime` (기본 0): 이 시간 동안은 데이터를 "신선하다"고 보고 다시 안 가져온다. 지나면 stale — 캐시를 일단 보여주되, 마운트/포커스 시 백그라운드로 재요청한다. **"낡은 걸 먼저 보여주고 뒤에서 갱신"**(stale-while-revalidate)이 기본 철학.
- `gcTime` (기본 5분): 사용하는 컴포넌트가 하나도 없어진 뒤, 캐시를 메모리에서 지우기까지의 유예.

**invalidation과 낙관적 업데이트**: 변경(mutation) 후 관련 쿼리를 `invalidateQueries`로 낡음 처리하는 게 기본. 더 즉각적인 UX가 필요하면 낙관적 업데이트 — 서버 응답을 기다리지 않고 캐시를 먼저 고치고, 실패하면 `onError`에서 이전 스냅샷으로 롤백한다.

**서버 컴포넌트 시대에 언제 여전히 필요한가**: 초기 데이터는 서버 컴포넌트가 담당한다(03장). TanStack Query가 남는 자리는 **클라이언트에서 데이터가 계속 살아 움직이는 경우** — 폴링(`refetchInterval`), 무한스크롤(`useInfiniteQuery`), 낙관적 업데이트, 탭 포커스 시 자동 갱신. "한 번 보여주고 끝"이면 서버 컴포넌트, "머무는 동안 계속 동기화"면 Query.

**Next.js와의 조합**: 서버에서 `queryClient.prefetchQuery`로 미리 채우고 `<HydrationBoundary state={dehydrate(queryClient)}>`로 넘기면, 첫 화면은 서버 렌더링의 속도를, 이후엔 Query의 동기화 능력을 둘 다 얻는다. 세부는 공식 가이드 참고: https://tanstack.com/query/latest/docs/framework/react/guides/advanced-ssr

### 클라이언트 상태 — 가벼운 것부터

- **`useState` + lifting state up**: 대부분은 이걸로 끝난다. 상태를 쓰는 컴포넌트들의 가장 가까운 공통 부모에 두는 것이 기본기.
- **Context는 주입용이지 상태 관리용이 아니다**: Context 값이 바뀌면 구독하는 모든 컴포넌트가 리렌더된다. 테마·로케일처럼 "거의 안 바뀌는 값의 주입"에 적합하고, 자주 바뀌는 상태를 넣으면 리렌더 폭탄이 된다.
- **Zustand — 전역이 정말 필요할 때**: 트리상 멀리 떨어진 컴포넌트들이 같은 상태를 자주 읽고 쓸 때. 선택적 구독이 핵심 — 스토어의 일부만 골라 구독하므로 관련 없는 변경에 리렌더되지 않는다.

```ts
const useUIStore = create<{ theme: 'light' | 'dark'; toggle: () => void }>((set) => ({
  theme: 'light',
  toggle: () => set((s) => ({ theme: s.theme === 'light' ? 'dark' : 'light' })),
}))
// 컴포넌트에서: const theme = useUIStore((s) => s.theme)  ← theme 바뀔 때만 리렌더
```

- **URL 상태**: 검색어·필터·페이지처럼 "링크로 공유했을 때 같은 화면이 나와야 하는" 상태는 `useState`가 아니라 URL이 원본이어야 한다. `useSearchParams`로 읽고 `router.replace`로 쓴다. 덤: 서버 컴포넌트도 `searchParams`로 같은 값을 읽을 수 있다.

### 판단 플로우차트

```
이 상태, 어디에 둘까?
│
├─ 원본이 서버 DB에 있는 데이터인가?
│   ├─ Yes → 화면에 한 번 뿌리면 끝인가?
│   │   ├─ Yes → 서버 컴포넌트에서 fetch (03장)
│   │   └─ No (폴링/무한스크롤/낙관적 업데이트/실시간 갱신)
│   │        → TanStack Query (+ 서버 prefetch 조합)
│   └─ No ↓
├─ URL로 공유/북마크/새로고침 시 유지돼야 하는가? (검색어, 필터, 페이지)
│   └─ Yes → URL 상태 (searchParams)
├─ 한 컴포넌트(또는 가까운 부모-자식)만 쓰는가? (모달, 입력값, 토글)
│   └─ Yes → useState (필요하면 lifting)
└─ 멀리 떨어진 여러 컴포넌트가 자주 읽고 쓰는가? (전역 UI 설정, 장바구니)
    └─ Yes → Zustand (거의 안 바뀌는 주입 값이면 Context)
```

위에서부터 순서대로 묻는 것이 요령이다 — 전역 스토어는 마지막 보루지 기본값이 아니다.

## 실습 (practice 앱에서)

- [ ] 메모 목록을 TanStack Query로 조회 + 삭제를 낙관적 업데이트로 (삭제 즉시 사라지고 실패 시 롤백)
  → **의미**: invalidation과 낙관적 업데이트가 서버 상태 관리의 본질(캐시 동기화)임을 체감하는 실습. 서버를 일부러 실패시켜 롤백까지 봐야 완성. 02-supabase 04장의 실시간 구독과 비교하면 "동기화의 두 방식"이 보인다.
- [ ] 메모 필터(전체/즐겨찾기)와 검색어를 URL 상태로 구현 — 새로고침/공유해도 유지되는지 확인
  → **의미**: "이 상태의 원본은 어디인가"라는 질문의 답이 useState가 아닐 수 있음을 확인. URL이 원본이면 서버 컴포넌트도 같은 값을 읽어 SSR(Server-Side Rendering)할 수 있다는 점이 Next.js에서 특히 중요하다.
- [ ] 다크 모드 같은 전역 UI 상태 하나를 Zustand로 관리해보기
  → **의미**: 선택적 구독으로 리렌더가 최소화되는 것을 React DevTools로 확인. 07장에서 next-themes로 했던 것과 비교해 "라이브러리가 해주던 일"의 정체를 파악한다.

## 참고 자료

- https://tanstack.com/query/latest
- https://zustand.docs.pmnd.rs
