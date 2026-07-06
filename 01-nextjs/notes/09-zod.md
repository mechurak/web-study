# 09. zod — 스키마로 검증하고 타입까지 얻기

> **핵심 질문**
> - TypeScript 타입은 런타임에 사라진다 — 그래서 무엇이 문제인가? (외부에서 들어오는 데이터는 못 믿는다)
> - "스키마 하나로 검증 + 타입 추론"이 왜 강력한가? (`z.infer`)
> - 검증은 어디에서 해야 하는가? — 경계(boundary)마다: 폼 입력, API 요청/응답, 환경변수

## 개념 정리

### 기본기
- [ ] 스키마 정의: `z.object`, `z.string().email()`, `z.number().min()`…
- [ ] `parse` vs `safeParse` — 예외 vs 결과 객체
- [ ] `z.infer<typeof schema>` — 스키마에서 타입 뽑기 (타입을 두 번 안 쓴다)
- [ ] transform, refine — 변환과 커스텀 규칙

### 경계마다 검증하기 (이 스택에서의 사용처 지도)
- [ ] **폼**: react-hook-form + `zodResolver` (07-shadcn의 Form 패턴)
- [ ] **Server Action**: FormData를 스키마로 검증 — Server Action도 공개 엔드포인트다
- [ ] **Route Handler / 외부 API 응답**: `await res.json()`은 `any`다 — 파싱 후 검증
- [ ] **환경변수**: 부팅 시 스키마 검증 (03-nestjs 05장, `@t3-oss/env-nextjs` 참고)
- [ ] **NestJS**: class-validator 대신 zod를 쓰는 선택지 (`nestjs-zod`) — 트레이드오프 정리

### 스키마 공유 — 프론트와 백엔드가 같은 진실을 본다
- [ ] Create/Update 스키마 파생 (`.partial()`, `.omit()`, `.pick()`)
- [ ] 프론트 폼 검증과 백엔드 API 검증에 **같은 스키마** 사용
- [ ] 공유 방법은 모노레포 구성이 필요 → [05-capstone/docs/monorepo.md](../../05-capstone/docs/monorepo.md)

## 실습 (practice 앱에서)

- [ ] 메모 스키마(`memoSchema`) 정의, Create/Update를 파생으로 생성
- [ ] 07장 shadcn 폼 + Server Action 양쪽에서 같은 스키마로 검증 (클라에서 통과, 서버 우회 요청은 거절되는 것 확인)
- [ ] 외부 API 응답을 zod로 파싱해서 필드 오타를 런타임에 잡아보기

## 참고 자료

- https://zod.dev
