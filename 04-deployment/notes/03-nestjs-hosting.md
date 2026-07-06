# 03. NestJS 호스팅 — 어디에 올릴 것인가

> **핵심 질문**
> - Vercel에 NestJS를 올리는 게 왜 부적합한가? (serverless vs 상시 프로세스)
> - PaaS(Platform as a Service — Railway/Fly.io/Render) vs 컨테이너(Cloud Run) vs VM — 추상화 수준의 트레이드오프는?
> - Dockerfile을 직접 쓰는 것의 가치는? (플랫폼 이식성)

## 개념 정리

### 선택지 비교
| 플랫폼 | 방식 | 장점 | 단점 | 무료 티어 |
|---|---|---|---|---|
| Railway | PaaS | (채우기) | | |
| Fly.io | 컨테이너 PaaS | | | |
| Render | PaaS | | | |
| Cloud Run | serverless 컨테이너 | | | |
| EC2/VM | IaaS(Infrastructure as a Service) | | | |

- [ ] 표 채우면서 하나 선택 (학습용 추천: Railway 또는 Fly.io — 마찰 최소)

### 컨테이너화
- [ ] NestJS용 Dockerfile 작성 (multi-stage build: 빌드 → 경량 런타임)
- [ ] `.dockerignore`, 이미지 크기 최적화
- [ ] 로컬에서 `docker build && docker run`으로 검증

### 배포 후 필수 확인
- [ ] `/health` 응답, 환경변수 주입, Supabase pooler 연결
- [ ] CORS: Vercel 도메인(프리뷰 포함) 허용 전략
- [ ] 프로세스 죽었을 때 자동 재시작 정책

## 실습

- [ ] 03-nestjs practice 서버를 Dockerfile로 컨테이너화, 로컬 검증
- [ ] 선택한 플랫폼에 배포, 공개 URL 확보
- [ ] Vercel의 Next.js 앱이 배포된 Nest API를 호출하도록 환경변수 연결
- [ ] 전체 경로 검증: 브라우저 → Vercel → Nest 호스팅 → Supabase

## 참고 자료

- https://docs.nestjs.com/deployment
