# 10. 프론트엔드 테스트 — Playwright E2E, React Testing Library

> **핵심 질문**
> - 백엔드 테스트(03-nestjs 05장)와 달리 프론트 테스트는 무엇을 검증하는가? — "유저가 보는 동작"
> - E2E와 컴포넌트 테스트의 비용/신뢰도 트레이드오프는? (테스팅 피라미드 vs 트로피)
> - "구현이 아니라 동작을 테스트하라"는 말의 의미는? (클래스명이 아니라 role/텍스트로 찾기)

## 개념 정리

### 테스트 층위 정하기
- [ ] E2E (Playwright): 실제 브라우저로 핵심 시나리오 — 적지만 신뢰도 최고
- [ ] 컴포넌트 (React Testing Library + Vitest): 로직 있는 컴포넌트 위주
- [ ] 무엇을 테스트 **안 할지** 정하기 (스냅샷 남발, 단순 표시 컴포넌트)

### Playwright
- [ ] 셋업, `page.goto` / `getByRole` / `expect(locator)`
- [ ] 인증 상태 재사용 (`storageState`) — 매 테스트 로그인 안 하기
- [ ] 테스트용 데이터 전략: 테스트 전용 Supabase 유저/데이터 준비와 정리
- [ ] trace viewer로 실패 디버깅

### React Testing Library
- [ ] 쿼리 우선순위: `getByRole` > `getByLabelText` > … > `getByTestId`
- [ ] `userEvent`로 상호작용, 비동기 `findBy*`
- [ ] 서버 컴포넌트는 RTL로 못 돌린다 — 클라이언트 컴포넌트/훅 위주로

### CI 연결
- [ ] GitHub Actions에서 Playwright 실행 (04-deployment 04장 CI 파이프라인에 추가)
- [ ] Vercel 프리뷰 배포 URL을 대상으로 E2E 돌리기 — 실배포 환경 검증

## 실습 (practice 앱에서)

- [ ] Playwright 셋업, 핵심 시나리오 E2E: 로그인 → 메모 작성 → 목록에 보임 → 삭제
- [ ] 인증 storageState 재사용 설정
- [ ] 메모 폼 컴포넌트를 RTL로 테스트 (검증 에러 표시 확인)
- [ ] CI에서 E2E가 돌고, 깨지면 머지가 막히는 것까지 확인

## 참고 자료

- https://playwright.dev/docs/intro
- https://testing-library.com/docs/react-testing-library/intro
