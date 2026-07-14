# 04. 스토리지 & 실시간

> **핵심 질문**
> - 파일은 왜 DB에 안 넣고 객체 스토리지에 넣는가?
> - 실시간 구독은 어떤 원리로 동작하는가? (Postgres replication → WebSocket)
> - 폴링 vs 실시간 구독 — 각각 언제 적합한가?

## 개념 정리

### 파일은 왜 DB에 안 넣는가

Postgres에 바이너리를 넣을 수는 있다(`bytea`). 안 하는 이유:

- **DB는 비싸고 스토리지는 싸다.** DB의 디스크·백업·복제는 전부 고비용 경로다. 5MB 이미지 수만 장이 백업마다 통째로 복사되는 것을 원하지 않는다.
- **서빙 경로가 다르다.** 이미지는 CDN(Content Delivery Network)이 캐시해서 뿌리는 게 맞고, DB 커넥션은 그 용도로 만들어지지 않았다. 파일 전송이 커넥션을 오래 점유하면 정작 쿼리가 대기한다.

그래서 관례는: **파일 본체는 객체 스토리지에, DB에는 경로(문자열)만.** Supabase Storage는 S3 호환 객체 스토리지에 Postgres 기반 권한 계층을 얹은 것이다 — 파일 메타데이터가 `storage.objects` 테이블에 있어서, 접근 제어를 우리가 이미 아는 RLS 정책으로 한다.

### 버킷 — public vs private

| | public 버킷 | private 버킷 |
|---|---|---|
| URL | 고정 공개 URL — 아는 사람은 누구나 접근 | 직접 접근 불가 |
| 접근 방법 | `getPublicUrl()` | **signed URL** — 만료 시간이 박힌 임시 서명 URL |
| 용도 | 아무나 봐도 되는 것 (게시물 이미지, 로고) | 권한이 필요한 것 (개인 문서, 비공개 프로필) |

```ts
await supabase.storage.from('avatars').upload(`${user.id}/avatar.png`, file)

// private 버킷: 1시간짜리 임시 URL 발급
const { data } = await supabase.storage
  .from('avatars').createSignedUrl(`${user.id}/avatar.png`, 3600)
```

- **Storage 정책**: 업로드/다운로드 권한도 policy로 건다. `storage.objects`에 대한 RLS라서 03장 문법 그대로다. 관례는 경로 첫 세그먼트를 유저 id로 쓰고 `(storage.foldername(name))[1] = auth.uid()::text`로 "자기 폴더만" 허용하는 패턴.
- **이미지 변환**: URL 파라미터로 리사이징·포맷 변환을 요청할 수 있다(`transform: { width: 100 }`). 썸네일을 미리 만들어 저장할 필요가 없어진다. (로컬 colima 환경에서는 image transformation이 안 뜰 수 있음 → 00장 §2.)

### Realtime — 원리: WAL → WebSocket

Postgres는 모든 변경을 **WAL(Write-Ahead Log)**이라는 로그에 먼저 쓴다(장애 복구용). 복제(replication)는 이 로그를 다른 서버가 구독해서 따라가는 것인데, Supabase Realtime은 **복제본인 척 WAL을 구독하는 서버**다. 테이블 변경이 WAL에 찍히면 → Realtime 서버가 읽어서 → WebSocket으로 구독 중인 브라우저들에 쏜다. 앱 코드가 "변경을 알리는" 일을 전혀 안 해도 되는 이유 — DB에 쓰는 것 자체가 알림이다.

세 가지 기능이 한 WebSocket 위에 있다:

- **Postgres Changes** — 위 원리로 테이블의 INSERT/UPDATE/DELETE를 구독. **RLS가 여기도 적용된다** — 내가 볼 수 없는 행의 변경은 이벤트도 안 온다.
- **Broadcast** — DB를 거치지 않고 클라이언트끼리 메시지 중계. 저장할 필요 없는 순간 데이터(커서 위치, 타이핑 표시).
- **Presence** — "지금 누가 접속해 있나" 상태 공유(온라인 표시, 동시 편집자 목록).

### 클라이언트 구독 패턴 — 구독은 반드시 해제와 짝

```tsx
'use client'
useEffect(() => {
  const channel = supabase
    .channel('memos-changes')
    .on('postgres_changes',
      { event: '*', schema: 'public', table: 'memos' },
      (payload) => { /* 목록 갱신 */ })
    .subscribe()

  return () => { supabase.removeChannel(channel) }   // 정리 필수
}, [])
```

해제를 빼먹으면 컴포넌트가 리마운트될 때마다 구독이 쌓여 같은 이벤트가 중복 도착한다 — React `useEffect` 정리 함수의 교과서적 사용처(01-nextjs 08장의 클라이언트 상태 지식과 연결).

### 폴링 vs 실시간 구독

| | 폴링 (주기적 재조회) | Realtime 구독 |
|---|---|---|
| 원리 | n초마다 다시 fetch | 변경이 생길 때만 push |
| 지연 | 주기만큼 늦음 | 즉시 |
| 비용 | 변경 없어도 매번 쿼리 | 유휴 시 거의 0, 대신 WebSocket 연결 유지 |
| 구현 | 단순, 상태 없음 | 구독 관리·재연결 처리 필요 |
| 적합 | 갱신이 드물고 지연 허용 (대시보드 통계) | 여러 사용자가 같은 데이터를 동시에 보는 화면 (채팅, 협업, 알림) |

기본값은 폴링(또는 그냥 페이지 이동 시 재조회)이어도 된다는 것이 중요하다. 실시간은 "즉시성"이 제품 가치일 때 선택하는 것이지 기본 사양이 아니다 — 연결 수가 곧 리소스이기 때문이다.

## 실습 (01-nextjs practice 앱에)

- [ ] `avatars` private 버킷 생성(+ 자기 폴더만 쓰기 가능한 storage 정책) → 프로필 이미지 업로드 → signed URL로 표시
  → **의미**: "파일은 스토리지에, DB엔 경로만" 구조와 signed URL의 만료를 직접 본다. storage 정책이 03장 RLS 문법 그대로임을 확인하는 것도 포인트 — 배운 모델이 재사용된다.
- [ ] 메모 목록을 실시간 구독으로 전환 — 브라우저 창 두 개를 나란히 놓고 한쪽에서 작성하면 다른 쪽에 즉시 나타나는지 확인
  → **의미**: WAL→WebSocket 경로를 체감한다. "쓰기만 했는데 다른 클라이언트가 아는" 경험이 실시간 아키텍처의 감을 만든다. 05-capstone에서 실시간 기능을 넣을지 판단할 때의 기준이 된다.
- [ ] 구독 해제(`removeChannel`)를 일부러 빼고 페이지를 왔다갔다 하며 이벤트가 중복 도착하는 것 확인 → 정리 함수 복구
  → **의미**: 실시간 코드의 최다 빈출 버그를 미리 밟는다. useEffect 정리 함수가 "규칙이라서"가 아니라 "안 하면 이렇게 되니까" 필요함을 남긴다.

## 참고 자료

- https://supabase.com/docs/guides/storage
- https://supabase.com/docs/guides/realtime
- https://supabase.com/docs/guides/realtime/postgres-changes — RLS와 구독의 관계 포함
