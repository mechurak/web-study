# 02. REST API — DTO, 검증(Pipe), 예외 처리

> **핵심 질문**
> - DTO(Data Transfer Object)를 왜 따로 정의하는가? 엔티티를 그대로 받고/반환하면 무슨 문제가 생기는가?
> - 입력 검증을 컨트롤러 코드가 아닌 선언(데코레이터)으로 하면 무엇이 좋은가?
> - 일관된 에러 응답 형식이 프론트엔드에 왜 중요한가?

## 개념 정리

### REST API 설계
- [ ] 리소스 중심 URL, HTTP 메서드/상태 코드의 올바른 사용
- [ ] `@Get/@Post/@Param/@Query/@Body` 데코레이터

### DTO와 검증
- [ ] `class-validator` + `class-transformer` (`@IsString`, `@IsOptional`…)
- [ ] 전역 `ValidationPipe` (`whitelist`, `transform` 옵션의 의미)
- [ ] Create DTO / Update DTO 분리 (`PartialType`)

### 예외 처리
- [ ] 내장 예외 (`NotFoundException`, `BadRequestException`…)
- [ ] Exception Filter로 에러 응답 형식 통일
- [ ] 예상 못한 에러 로깅

### API 문서화
- [ ] `@nestjs/swagger` — 데코레이터에서 OpenAPI 문서 자동 생성

## 실습 (practice 서버에)

- [ ] 메모 CRUD(Create, Read, Update, Delete)를 REST(Representational State Transfer) 규칙에 맞게 구현 (적절한 상태 코드 포함)
- [ ] CreateMemoDto에 검증 걸고, 잘못된 요청이 400으로 떨어지는 것 확인
- [ ] Swagger UI (`/api`) 띄워서 문서 확인
- [ ] Next.js 앱에서 이 API를 호출하도록 연결 (CORS 설정 포함)

## 참고 자료

- https://docs.nestjs.com/techniques/validation
