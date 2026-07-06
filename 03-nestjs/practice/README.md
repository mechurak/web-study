# NestJS 실습 서버

이 폴더에 실습용 NestJS API 서버 **하나**를 만들고, 각 노트 챕터의 실습을 모듈 단위로 추가해나간다.

## 시작하기

```bash
cd 03-nestjs/practice
npx @nestjs/cli new api
cd api
npm run start:dev
```

## 챕터별 실습 진행 상황

- [ ] 01 아키텍처: memos 모듈 (in-memory)
- [ ] 02 REST(Representational State Transfer) API: DTO(Data Transfer Object) 검증 + 예외 처리 + Swagger + Next.js 연동(CORS)
- [ ] 03 DB: Prisma로 Supabase Postgres 연결, 트랜잭션
- [ ] 04 인증: Supabase JWT(JSON Web Token) 검증 Guard, 소유권 검사
- [ ] 05 테스트/설정: 단위+e2e(End-to-End) 테스트, ConfigModule, /health

> 완성되면 Next(프론트) ↔ Nest(백엔드) ↔ Supabase(DB) 3층 구조가 된다.
> 04-deployment에서 이 서버를 실제로 호스팅한다.
