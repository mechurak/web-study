# 05. 테스트 & 설정 관리

> **핵심 질문**
> - 단위 테스트 / 통합 테스트 / e2e(End-to-End) 테스트의 경계는? 각각 무엇을 믿게 해주는가?
> - DI(Dependency Injection, 의존성 주입) 덕분에 테스트가 쉬워진다는 게 구체적으로 무슨 뜻인가? (모킹)
> - 환경변수를 코드에서 바로 `process.env`로 읽으면 무엇이 문제인가?

## 개념 정리

### 테스트
- [ ] Jest 기본, `Test.createTestingModule`
- [ ] 서비스 단위 테스트 — 리포지토리/의존성 모킹
- [ ] 컨트롤러 e2e 테스트 (`supertest`) — 실제 HTTP 요청으로 검증
- [ ] 테스트용 DB 전략 (로컬 Supabase / 트랜잭션 롤백)

### 설정 관리
- [ ] `@nestjs/config` — `.env` 로딩, `ConfigService` 주입
- [ ] 환경별 설정 (local / production), `.env`는 커밋 금지 + `.env.example` 제공
- [ ] 시작 시 설정 검증 (Joi/zod 스키마) — 빠진 환경변수를 부팅 시점에 잡기

### 운영 준비
- [ ] 로깅 (내장 Logger, 요청 로깅 인터셉터)
- [ ] health check 엔드포인트 (`@nestjs/terminus`) — 배포 챕터에서 사용

## 실습 (practice 서버에)

- [ ] MemosService 단위 테스트 (DB 모킹)
- [ ] 메모 CRUD e2e 테스트 1개 이상
- [ ] ConfigModule + 환경변수 검증 적용, `.env.example` 작성
- [ ] `/health` 엔드포인트 추가

## 참고 자료

- https://docs.nestjs.com/fundamentals/testing
