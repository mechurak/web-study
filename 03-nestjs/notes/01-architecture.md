# 01. 아키텍처 — 모듈 / 컨트롤러 / 프로바이더, DI

> **핵심 질문**
> - 의존성 주입(DI)이란? `new`로 직접 만들지 않고 주입받으면 무엇이 좋은가? (테스트, 교체 가능성)
> - 모듈은 무엇을 묶는 단위인가? `imports/providers/exports`는 각각 무슨 의미인가?
> - 요청 하나가 컨트롤러 → 서비스 → (리포지토리)로 흐르는 레이어드 구조의 이유는?

## 개념 정리

### 3대 구성 요소
- [ ] Module (`@Module`) — 기능 단위 묶음, 의존성 그래프의 노드
- [ ] Controller (`@Controller`) — HTTP 요청 받기 (라우팅 + 파라미터 추출만)
- [ ] Provider/Service (`@Injectable`) — 비즈니스 로직

### 의존성 주입
- [ ] 생성자 주입 패턴
- [ ] 프로바이더 스코프 (기본 싱글톤)
- [ ] 커스텀 프로바이더 (`useValue`, `useFactory`) — 언제 쓰나

### 요청 생명주기
- [ ] 미들웨어 → 가드 → 인터셉터 → 파이프 → 컨트롤러 → 인터셉터 → 필터
- [ ] 각 단계의 용도 한 줄씩 정리

### CLI
- [ ] `nest g module/controller/service` 로 스캐폴딩

## 실습 (practice 서버에)

- [ ] `nest new`로 프로젝트 생성, 구조 파악
- [ ] `memos` 모듈/컨트롤러/서비스 생성 (일단 in-memory 배열로)
- [ ] 서비스를 컨트롤러에 주입하고, DI 없이 짰을 때와 테스트 용이성 비교해보기

## 참고 자료

- https://docs.nestjs.com/first-steps
