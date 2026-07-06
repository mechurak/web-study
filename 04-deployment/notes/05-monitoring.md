# 05. 모니터링 — 로그, 애널리틱스, 에러 추적

> **핵심 질문**
> - 배포 후 "서비스가 잘 돌고 있다"를 무엇으로 판단하는가? (에러율, 응답 시간, 가용성)
> - console.log와 구조화된 로깅의 차이는? 로그는 어디에 쌓이고 언제 사라지는가?
> - 유저가 겪은 에러를 유저가 신고하기 전에 알 수 있는 방법은?

## 개념 정리

### 로그
- [ ] Vercel 함수 로그 / 호스팅 플랫폼 로그 보는 법
- [ ] 구조화된 로깅 (JSON), 요청 ID로 프론트-백엔드 로그 연결하기
- [ ] 로그 보존 기간 문제와 외부 수집 (필요해지면)

### 에러 추적
- [ ] Sentry — 프론트(Next)와 백엔드(Nest) 모두 연동
- [ ] 소스맵, 릴리즈 태깅, 알림 설정
- [ ] 에러 그룹핑과 우선순위 판단

### 성능/사용 지표
- [ ] Vercel Analytics / Speed Insights — Core Web Vitals 실측
- [ ] uptime 모니터링 (`/health` 주기 호출 — UptimeRobot 등)
- [ ] Supabase 대시보드: 쿼리 성능, 커넥션 수

## 실습

- [ ] Sentry를 Next + Nest에 연동, 일부러 에러 발생시켜 알림까지 확인
- [ ] Vercel Speed Insights 켜고 실사용 Web Vitals 확인
- [ ] uptime 모니터가 Nest `/health`를 감시하게 설정

## 참고 자료

- https://vercel.com/docs/observability
