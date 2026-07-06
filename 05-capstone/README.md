# 05. Capstone — 통합 실전 프로젝트

앞의 모든 챕터를 관통하는 작은 서비스를 **요구사항 정의부터 배포·모니터링까지** 처음부터 끝까지 만든다.

## 주제 (정하기)

후보 — 전 레이어를 조금씩 다 쓰는 규모가 적당하다:

- [ ] **북마크/링크 공유 서비스** — CRUD + 태그 + 공개/비공개(RLS) + OG 미리보기(백엔드 크롤링)
- [ ] **팀 TODO / 칸반** — 실시간 동기화 + 멤버 초대(인가 로직) + 활동 로그
- [ ] **간단 블로그 플랫폼** — SSG/ISR 활용 + 이미지 업로드 + SEO
- [ ] 직접 정한 주제: ___________

선정 기준: 인증 필요 / DB 관계 2~3개 / 백엔드 로직이 자연스럽게 필요한 기능 1개 이상 / 2~4주 규모

## 아키텍처

```
브라우저 ── Vercel (Next.js) ──┬── Supabase (인증, 단순 CRUD는 RLS로 직접)
                               └── Nest API (복잡한 로직) ── Supabase Postgres
```

## 진행 단계

| 단계 | 산출물 | 상태 |
|---|---|---|
| 0. 모노레포 셋업 | [docs/monorepo.md](./docs/monorepo.md) — pnpm workspaces, `packages/shared` | ⬜ |
| 1. 요구사항 정의 | `docs/requirements.md` — 유저 스토리, 기능 목록, 범위 밖 명시 | ⬜ |
| 2. 설계 | `docs/design.md` — 화면 흐름, DB 스키마(ERD), API 명세, Next↔Nest 역할 분담 | ⬜ |
| 3. DB 구축 | 마이그레이션 + RLS 정책 | ⬜ |
| 4. 백엔드 구현 | `backend/` — NestJS | ⬜ |
| 5. 프론트 구현 | `frontend/` — Next.js | ⬜ |
| 6. 배포 | Vercel + Nest 호스팅 + 도메인 + CI | ⬜ |
| 7. 운영 | 모니터링 연동, 배포 후 회고 `docs/retrospective.md` | ⬜ |

## 폴더 구조

```
05-capstone/
├── README.md            # 이 파일 — 진행 상황 추적
├── docs/                # 요구사항, 설계, 회고, 모노레포 가이드
├── pnpm-workspace.yaml  # (셋업 시 생성)
├── packages/shared/     # zod 스키마, 공유 타입 (frontend/backend 공용)
├── frontend/            # Next.js (Vercel 배포)
└── backend/             # NestJS (Supabase 연결)
```
