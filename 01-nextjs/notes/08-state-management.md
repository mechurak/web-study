# 08. 상태 관리 — 서버 상태 vs 클라이언트 상태

> **핵심 질문**
> - "서버 상태"와 "클라이언트 상태"는 무엇이 다른가? 왜 다른 도구로 관리하는가?
> - 어떤 데이터가 생겼을 때, 서버 컴포넌트 / TanStack Query / useState / Zustand 중 무엇으로 다룰지 판단하는 기준은?
> - Redux가 표준이던 시대에서 왜 이렇게 바뀌었는가? (서버 상태를 전역 스토어에 넣던 시절의 문제)

## 개념 정리

### 상태의 분류부터
- [ ] 서버 상태: DB가 원본, 캐시일 뿐 (메모 목록, 유저 프로필)
- [ ] 클라이언트 상태: 브라우저가 원본 (모달 열림, 선택된 탭, 폼 입력 중)
- [ ] URL 상태: 검색어, 필터, 페이지 번호 — searchParams가 원본인 것들

### 서버 상태 — TanStack Query
- [ ] `useQuery` / `useMutation`, queryKey 설계
- [ ] staleTime / gcTime — "신선함"의 개념
- [ ] invalidation, 낙관적 업데이트(optimistic update)
- [ ] 서버 컴포넌트 시대에 언제 여전히 필요한가: 폴링, 무한스크롤, 클라이언트 주도 갱신
- [ ] Next.js와의 조합: 서버에서 prefetch → `HydrationBoundary`로 넘기기

### 클라이언트 상태 — 가벼운 것부터
- [ ] `useState` / lifting state up — 대부분은 이걸로 충분
- [ ] Context — 주입용이지 상태 관리용이 아니다 (리렌더 전파 문제)
- [ ] Zustand — 전역이 정말 필요할 때 (스토어 정의, 선택적 구독)
- [ ] URL 상태: `useSearchParams` + `router.replace` — 공유 가능한 상태는 URL로

### 판단 플로우차트 만들기
- [ ] "이 상태는 어디에?" 결정 트리를 직접 그려서 이 노트에 남기기

## 실습 (practice 앱에서)

- [ ] 메모 목록을 TanStack Query로 조회 + 삭제를 낙관적 업데이트로 (삭제 즉시 사라지고 실패 시 롤백)
- [ ] 메모 필터(전체/즐겨찾기)와 검색어를 URL 상태로 구현 — 새로고침/공유해도 유지되는지 확인
- [ ] 다크 모드 같은 전역 UI 상태 하나를 Zustand로 관리해보기

## 참고 자료

- https://tanstack.com/query/latest
- https://zustand.docs.pmnd.rs
