# 01. Vercel — 동작 원리와 배포 흐름

> **핵심 질문**
> - Vercel에 Next.js를 올리면 각 부분(정적 페이지, SSR(Server-Side Rendering), API Route)이 실제로 어떤 인프라에서 실행되는가?
> - "서버가 없다(serverless)"는 말의 실제 의미는? cold start란?
> - 프리뷰 배포가 개발 워크플로우를 어떻게 바꾸는가?

## 개념 정리

### Vercel이 Next.js를 배포하는 방식
- [ ] 정적 자산/SSG(Static Site Generation) 페이지 → CDN(Content Delivery Network)
- [ ] SSR/Route Handler → Serverless Function (실행 시간·메모리 제한)
- [ ] 미들웨어 → Edge Runtime (전 세계 엣지에서 실행)
- [ ] ISR(Incremental Static Regeneration)은 어떻게 구현되는가

### 배포 흐름
- [ ] Git 연동: push → 자동 빌드 → 배포
- [ ] Production / Preview / Development 환경 구분
- [ ] 프리뷰 배포 — PR마다 고유 URL, 리뷰 문화와의 연결
- [ ] 롤백 — 이전 배포로 즉시 되돌리기 (불변 배포의 장점)

### 한계 인지
- [ ] 함수 실행 시간 제한 → 장시간 작업 불가
- [ ] WebSocket 상시 연결 불가 → NestJS를 딴 데 두는 이유
- [ ] 비용 구조 (무료 티어 범위, 함수 호출량 과금)

## 실습

- [ ] 01-nextjs practice 앱을 GitHub에 올리고 Vercel 연결
- [ ] 프로덕션 배포 + 브랜치 만들어 프리뷰 배포 확인
- [ ] 빌드 로그에서 어떤 페이지가 static/λ(function)인지 확인
- [ ] 일부러 실패하는 커밋 → 배포가 막히는 것 확인 → 롤백 해보기

## 참고 자료

- https://vercel.com/docs/frameworks/nextjs
