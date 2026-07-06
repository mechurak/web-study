# 03. 데이터 페칭 — fetch 캐싱, Server Actions

> **핵심 질문**
> - Next.js의 `fetch`는 브라우저 fetch와 뭐가 다른가? (캐싱, 중복 제거)
> - Server Action이란? API Route 없이 어떻게 서버 코드를 호출하는가?
> - 언제 Server Action을 쓰고, 언제 별도 API(Route Handler / NestJS)를 쓰는가?

## 개념 정리

### 서버 컴포넌트에서의 데이터 페칭
- [ ] `async` 서버 컴포넌트에서 직접 `await fetch()`
- [ ] 캐싱 옵션: `cache: 'force-cache' | 'no-store'`, `next: { revalidate }`
- [ ] 같은 요청의 자동 중복 제거 (request deduplication)

### 캐시 무효화
- [ ] `revalidatePath` / `revalidateTag`
- [ ] 시간 기반 vs 온디맨드 revalidation

### Server Actions
- [ ] `"use server"` — 폼 제출을 서버 함수로 직접 처리
- [ ] `useFormStatus`, `useActionState`로 로딩/에러 상태 다루기
- [ ] 보안 유의점: Server Action도 결국 공개 엔드포인트다 (검증 필수)

### 클라이언트 페칭
- [ ] SWR / TanStack Query는 언제 여전히 필요한가 (폴링, 무한스크롤, 낙관적 업데이트)

## 실습 (practice 앱에서)

- [ ] 외부 공개 API(예: JSONPlaceholder)를 서버 컴포넌트에서 fetch해서 목록 렌더링
- [ ] 메모 추가 폼을 Server Action으로 구현 (일단 메모리/파일 저장, 나중에 Supabase로 교체)
- [ ] `revalidatePath`로 추가 후 목록 갱신 확인

## 참고 자료

- https://nextjs.org/docs/app/building-your-application/data-fetching
