# 04. 스토리지 & 실시간

> **핵심 질문**
> - 파일은 왜 DB에 안 넣고 객체 스토리지에 넣는가?
> - 실시간 구독은 어떤 원리로 동작하는가? (Postgres replication → WebSocket)
> - 폴링 vs 실시간 구독 — 각각 언제 적합한가?

## 개념 정리

### Storage
- [ ] 버킷 생성, public vs private 버킷
- [ ] 업로드/다운로드 API, signed URL (private 파일의 임시 공개)
- [ ] Storage에도 RLS(Row Level Security)와 같은 정책이 있다 (storage policies)
- [ ] 이미지 변환 (리사이징) 기능

### Realtime
- [ ] Postgres Changes — 테이블 변경 구독 (INSERT/UPDATE/DELETE)
- [ ] Broadcast / Presence — DB 없이 메시지 전달, 접속자 표시
- [ ] 클라이언트에서 구독하고 해제하기 (`useEffect` 정리 패턴)

## 실습 (01-nextjs practice 앱에)

- [ ] 프로필 이미지 업로드 (private 버킷 + signed URL 표시)
- [ ] 메모 목록을 실시간 구독으로 전환 — 두 브라우저 창에서 즉시 반영 확인

## 참고 자료

- https://supabase.com/docs/guides/storage
- https://supabase.com/docs/guides/realtime
