# 모노레포 — pnpm workspaces (+ Turborepo)

> capstone 시작 전 준비 문서. `frontend/`(Next)와 `backend/`(Nest)가 타입·zod 스키마를 공유하려면 필요하다.

> **핵심 질문**
> - 모노레포 vs 멀티레포 — 무엇이 좋아지고 무엇이 복잡해지는가?
> - "프론트와 백엔드가 같은 타입을 본다"가 실제로 막아주는 버그는? (API 응답 형태 불일치)
> - Turborepo는 pnpm workspaces 위에 무엇을 더해주는가? (태스크 파이프라인, 캐싱)

## 개념 정리

### pnpm workspaces
- [ ] `pnpm-workspace.yaml`, 루트/패키지별 `package.json` 구조
- [ ] `workspace:*` 프로토콜로 내부 패키지 참조
- [ ] 공유 패키지 만들기: `packages/shared` (zod 스키마, 타입, 상수)
- [ ] 공유 패키지의 빌드 전략: tsc로 빌드 vs 소스 직접 참조 — 트레이드오프

### Turborepo (선택)
- [ ] `turbo.json` — 태스크 의존성 (`build`가 `^build`에 의존)
- [ ] 캐싱 — 안 바뀐 패키지는 다시 빌드 안 함
- [ ] 언제 도입하나: 패키지 2~3개면 pnpm만으로 충분할 수 있다

### 배포와의 관계
- [ ] Vercel의 모노레포 지원 — Root Directory 지정, 영향받은 앱만 빌드
- [ ] Nest 쪽 Dockerfile에서 workspace 의존성 처리 (`pnpm deploy` 또는 turbo prune)
- [ ] CI에서 바뀐 패키지만 테스트하기

## capstone 목표 구조

```
05-capstone/
├── pnpm-workspace.yaml
├── packages/
│   └── shared/          # zod 스키마, API 타입, 상수 (01-nextjs 09장의 스키마 공유 실현)
├── frontend/            # Next.js — shared 참조
└── backend/             # NestJS — shared 참조
```

## 실습 (capstone 시작 단계에서)

- [ ] 위 구조로 workspace 셋업, `packages/shared`에 메모 zod 스키마 배치
- [ ] frontend와 backend 양쪽에서 import되는 것 확인 — 스키마 한 곳 수정이 양쪽에 반영
- [ ] Vercel에 frontend만 배포되도록 Root Directory 설정
- [ ] backend Dockerfile이 shared를 포함해 빌드되는지 확인

## 참고 자료

- https://pnpm.io/workspaces
- https://turborepo.dev/docs
