# 04. 커스텀 도메인 & CI/CD

> **핵심 질문**
> - 도메인을 사서 서비스에 연결하기까지의 과정은? (DNS 레코드, A/CNAME, TLS 인증서)
> - CI와 CD는 각각 무엇을 자동화하는가? Vercel의 Git 연동만으로 부족한 지점은?
> - "main에 머지되면 자동으로 프로덕션 배포"가 안전하려면 그 앞에 무엇이 있어야 하는가?

## 개념 정리

### 도메인
- [ ] 도메인 구매 → 네임서버/DNS 레코드 설정 → Vercel 연결
- [ ] A 레코드 vs CNAME, TLS 인증서 자동 발급 (Let's Encrypt)
- [ ] 서브도메인 전략: `www`, `api.도메인` (Nest 서버용)

### CI — 머지 전 검증
- [ ] GitHub Actions 기본 구조 (workflow / job / step)
- [ ] PR마다: lint + typecheck + test 실행
- [ ] Playwright E2E도 CI에 포함 (01-nextjs 10장에서 작성한 것)
- [ ] 브랜치 보호 규칙 — CI 통과해야 머지 가능

### CD — 배포 자동화
- [ ] Vercel: Git 연동이 곧 CD (프리뷰 → 머지 시 프로덕션)
- [ ] Nest 쪽: 플랫폼 Git 연동 또는 GitHub Actions에서 배포 트리거
- [ ] 프론트/백엔드 배포 순서 문제 (API 먼저? 하위 호환?)

## 실습

- [ ] (선택) 도메인 하나 구입해서 Vercel 앱에 연결
- [ ] GitHub Actions로 PR CI 파이프라인 구성 (lint+test), 브랜치 보호 설정
- [ ] 일부러 테스트 깨는 PR을 만들어 머지가 막히는 것 확인

## 참고 자료

- https://vercel.com/docs/projects/domains
- https://docs.github.com/actions
