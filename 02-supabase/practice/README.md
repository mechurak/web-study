# Supabase 실습

실습 코드 자체는 **01-nextjs/practice 앱에 Supabase를 붙이는 방식**으로 진행한다.
이 폴더에는 앱 코드가 아닌 것들을 기록한다:

- `migrations/` — 테이블 생성/변경 SQL (Supabase CLI 마이그레이션)
- `policies.sql` — RLS 정책 모음 (마이그레이션에 포함해도 됨)
- `setup.md` — 프로젝트 생성, OAuth 앱 등록 등 대시보드에서 한 설정 기록

## 시작하기

```bash
# Supabase CLI 설치 및 로컬 개발 환경
brew install supabase/tap/supabase
supabase init
supabase start   # 로컬 Docker Postgres + Studio
```

## 진행 상황

- [ ] Supabase 프로젝트 생성 (클라우드 + 로컬)
- [ ] 01 테이블 설계: `memos`, `profiles` 마이그레이션
- [ ] 02 인증: 이메일 + GitHub OAuth
- [ ] 03 RLS: 본인 메모만 접근 정책
- [ ] 04 스토리지(프로필 이미지) + 실시간(메모 목록)
- [ ] 05 Next.js 연동 패턴 정리 완료
