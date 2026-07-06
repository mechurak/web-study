# 00. Overview — 웹 서비스 아키텍처 큰 그림

> **핵심 질문**
> - 브라우저에 URL을 입력하면 응답이 오기까지 무슨 일이 일어나는가? (DNS → CDN → 서버 → DB)
> - "프론트엔드 / 백엔드 / DB"의 경계는 어디인가? 요즘 스택에서는 왜 이 경계가 흐려지는가?
> - 이 레포의 스택(Next.js + Vercel + Supabase + NestJS)은 각각 어느 레이어를 담당하는가?

## 목차

### 1. 요청의 일생 (Life of a Request)
- [ ] DNS 조회 → CDN/Edge → 오리진 서버 → DB → 응답 렌더링 흐름 그리기
- [ ] 정적 파일 서빙 vs 서버 렌더링 vs API 호출의 차이

### 2. 레이어별 역할과 이 레포의 스택
- [ ] Frontend: Next.js — 화면 + 라우팅 + (일부) 서버 로직
- [ ] Backend: NestJS — 비즈니스 로직, 인증/인가, 외부 연동
- [ ] DB/BaaS: Supabase — Postgres + 인증 + 스토리지 + 실시간
- [ ] 배포: Vercel(프론트) + 별도 호스팅(백엔드)

### 3. 왜 이 스택인가 — 대안 비교
- [ ] Next.js vs SPA(Vite+React) vs Remix
- [ ] Supabase vs Firebase vs 직접 Postgres 운영
- [ ] NestJS vs Express vs Fastify vs "Next API Route로 다 하기"
- [ ] Vercel vs AWS/자체 서버

### 4. 아키텍처 다이어그램
- [ ] 최종 목표 아키텍처를 그림으로 정리 (capstone에서 실제로 구현할 형태)

## 정리 노트

(학습하며 여기에 채워나가기)
