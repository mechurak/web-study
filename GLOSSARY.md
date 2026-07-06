# 용어집 (Glossary)

레포 전체에서 쓰이는 주요 용어·약어 모음. 각 용어를 깊이 다루는 노트로 링크한다.
문서를 읽다 모르는 약어가 나오면 여기서 찾으면 된다.

## 웹 기초 & 인프라

| 용어 | 풀어 쓰면 | 뜻 | 자세히 |
|---|---|---|---|
| DNS | Domain Name System | 도메인 이름(myapp.com)을 IP 주소로 바꿔주는 시스템 | [00-overview](./00-overview/README.md) |
| TLS | Transport Layer Security | 통신 암호화 프로토콜. https의 자물쇠. (구명칭 SSL) | [00-overview](./00-overview/README.md) |
| CDN | Content Delivery Network | 전 세계에 분산된 캐시 서버망. 유저와 가까운 곳에서 응답해 빠르다 | [00-overview](./00-overview/README.md), [04-deployment 01](./04-deployment/notes/01-vercel.md) |
| 엣지 (Edge) | — | CDN의 각 지점. 캐시 서빙을 넘어 코드 실행(Edge Runtime)까지 하는 위치 | [01-nextjs 04](./01-nextjs/notes/04-api-routes.md) |
| 오리진 (Origin) | — | CDN 뒤에 있는 원본 서버. 캐시에 없으면 여기까지 온다 | [00-overview](./00-overview/README.md) |
| Serverless | — | 상시 서버 없이 요청이 올 때만 함수가 실행되는 모델. 실행 시간 제한, cold start가 특징 | [04-deployment 01](./04-deployment/notes/01-vercel.md) |
| Cold start | — | serverless 함수가 잠들어 있다가 첫 요청에 깨어나며 생기는 지연 | [04-deployment 01](./04-deployment/notes/01-vercel.md) |
| SPA | Single-Page Application | HTML 한 장 + JS로 화면을 전부 브라우저에서 그리는 방식. 첫 로딩·SEO가 약점 | [00-overview](./00-overview/README.md) |
| SEO | Search Engine Optimization | 검색 엔진 최적화. 크롤러가 내용을 읽을 수 있어야 검색에 노출된다 | [01-nextjs 05](./01-nextjs/notes/05-optimization.md) |
| OG | Open Graph | 링크 공유 시 보이는 미리보기(제목/이미지)를 정의하는 메타태그 규약 | [01-nextjs 05](./01-nextjs/notes/05-optimization.md) |
| PaaS / IaaS | Platform / Infrastructure as a Service | 추상화 수준에 따른 클라우드 분류. PaaS는 코드만 주면 돌려줌(Railway), IaaS는 VM부터 직접(EC2) | [04-deployment 03](./04-deployment/notes/03-nestjs-hosting.md) |

## 렌더링 & Next.js

| 용어 | 풀어 쓰면 | 뜻 | 자세히 |
|---|---|---|---|
| SSR | Server-Side Rendering | 요청 시마다 서버에서 HTML을 만들어 보내는 방식 | [01-nextjs 02](./01-nextjs/notes/02-rendering.md) |
| SSG | Static Site Generation | 빌드 시 HTML을 미리 만들어 CDN에서 서빙하는 방식 | [01-nextjs 02](./01-nextjs/notes/02-rendering.md) |
| ISR | Incremental Static Regeneration | SSG + 주기적 재생성. 정적의 속도와 "적당히 최신"을 동시에 | [01-nextjs 02](./01-nextjs/notes/02-rendering.md) |
| CSR | Client-Side Rendering | 브라우저에서 JS로 화면을 그리는 방식 (SPA의 기본 동작) | [01-nextjs 02](./01-nextjs/notes/02-rendering.md) |
| RSC | React Server Components | 서버에서만 실행되고 코드가 브라우저로 전송되지 않는 컴포넌트. App Router의 기본값 | [01-nextjs 02](./01-nextjs/notes/02-rendering.md) |
| Hydration | — | 서버가 보낸 정적 HTML에 브라우저에서 JS(이벤트, 상태)를 붙여 살아나게 하는 과정 | [01-nextjs 02](./01-nextjs/notes/02-rendering.md) |
| Server Action | — | API 엔드포인트 없이 폼 제출을 서버 함수로 직접 처리하는 Next.js 기능 (`"use server"`) | [01-nextjs 03](./01-nextjs/notes/03-data-fetching.md) |
| Route Handler | — | Next.js 안에서 JSON을 반환하는 API 엔드포인트 (`app/api/*/route.ts`) | [01-nextjs 04](./01-nextjs/notes/04-api-routes.md) |
| Prefetch | — | 링크가 뷰포트에 보이면 대상 페이지를 미리 받아두는 것. 클릭 시 즉시 전환의 비결 | [01-nextjs 01](./01-nextjs/notes/01-routing.md) |
| JIT | Just-in-Time | Tailwind가 소스에서 발견한 클래스만 골라 CSS를 생성하는 방식 | [01-nextjs 06](./01-nextjs/notes/06-tailwind.md) |
| CVA | Class Variance Authority | variant(색/크기별 스타일 조합)를 선언적으로 관리하는 라이브러리. shadcn의 구성 요소 | [01-nextjs 07](./01-nextjs/notes/07-shadcn.md) |

## 데이터 & 백엔드

| 용어 | 풀어 쓰면 | 뜻 | 자세히 |
|---|---|---|---|
| CRUD | Create, Read, Update, Delete | 데이터 다루기의 기본 4연산 | [01-nextjs 04](./01-nextjs/notes/04-api-routes.md), [03-nestjs 02](./03-nestjs/notes/02-rest-api.md) |
| REST | Representational State Transfer | 리소스 중심 URL + HTTP 메서드로 API를 설계하는 관례 | [03-nestjs 02](./03-nestjs/notes/02-rest-api.md) |
| SQL | Structured Query Language | 관계형 DB에 질의하는 표준 언어 | [02-supabase 01](./02-supabase/notes/01-postgres-basics.md) |
| ORM | Object-Relational Mapping | 테이블을 객체로 다루게 해주는 계층 (Prisma, TypeORM) | [03-nestjs 03](./03-nestjs/notes/03-database.md) |
| N+1 문제 | — | 목록 1번 + 항목마다 1번씩 쿼리가 나가는 ORM의 대표 성능 함정 | [03-nestjs 03](./03-nestjs/notes/03-database.md) |
| BaaS | Backend as a Service | DB·인증·스토리지 등 백엔드 기능을 서비스로 제공 (Supabase, Firebase) | [02-supabase](./02-supabase/README.md) |
| RLS | Row Level Security | Postgres의 행 단위 접근 제어. 클라이언트가 DB에 직접 쿼리해도 안전한 이유 | [02-supabase 03](./02-supabase/notes/03-rls.md) |
| JWT | JSON Web Token | 서명된 토큰으로 유저 신원을 증명하는 방식. Supabase 로그인 → NestJS 검증의 매개체 | [02-supabase 02](./02-supabase/notes/02-auth.md), [03-nestjs 04](./03-nestjs/notes/04-auth.md) |
| 인증 / 인가 | Authentication / Authorization | "누구인가"의 확인 vs "무엇을 할 수 있는가"의 판단. 다른 문제다 | [02-supabase 02](./02-supabase/notes/02-auth.md) |
| DI | Dependency Injection (의존성 주입) | 의존 객체를 직접 만들지 않고 주입받는 패턴. 테스트·교체가 쉬워진다. NestJS의 뼈대 | [03-nestjs 01](./03-nestjs/notes/01-architecture.md) |
| DTO | Data Transfer Object | 요청/응답의 형태를 정의하는 객체. 검증의 단위가 된다 | [03-nestjs 02](./03-nestjs/notes/02-rest-api.md) |
| Connection Pooler | — | DB 커넥션을 재사용하는 중계층(pgbouncer). serverless에서 커넥션 고갈을 막는다 | [03-nestjs 03](./03-nestjs/notes/03-database.md) |
| Webhook | — | 이벤트 발생 시 외부 서비스가 내 URL을 호출해주는 방식 (결제 알림 등) | [01-nextjs 04](./01-nextjs/notes/04-api-routes.md) |
| 트랜잭션 (Transaction) | — | 여러 DB 작업을 전부 성공 또는 전부 취소로 묶는 단위 | [03-nestjs 03](./03-nestjs/notes/03-database.md) |

## 품질 · 보안 · 운영

| 용어 | 풀어 쓰면 | 뜻 | 자세히 |
|---|---|---|---|
| CI / CD | Continuous Integration / Continuous Deployment | 머지 전 자동 검증 / 머지 후 자동 배포 | [04-deployment 04](./04-deployment/notes/04-domain-ci.md) |
| E2E | End-to-End | 실제 브라우저로 유저 시나리오 전체를 검증하는 테스트 (Playwright) | [01-nextjs 10](./01-nextjs/notes/10-testing.md) |
| RTL | React Testing Library | 컴포넌트를 유저 관점 쿼리(getByRole)로 테스트하는 라이브러리 | [01-nextjs 10](./01-nextjs/notes/10-testing.md) |
| Core Web Vitals | — | 구글의 사용자 경험 3지표: LCP(최대 콘텐츠 표시), CLS(레이아웃 밀림), INP(상호작용 반응) | [01-nextjs 05](./01-nextjs/notes/05-optimization.md) |
| XSS | Cross-Site Scripting | 남의 페이지에 악성 스크립트를 심는 공격 | [04-deployment 06](./04-deployment/notes/06-security.md) |
| CSRF | Cross-Site Request Forgery | 로그인된 유저의 브라우저를 속여 원치 않는 요청을 보내게 하는 공격 | [04-deployment 06](./04-deployment/notes/06-security.md) |
| CSP | Content Security Policy | 페이지가 로드/실행할 수 있는 리소스를 제한하는 보안 헤더 | [04-deployment 06](./04-deployment/notes/06-security.md) |
| OWASP | Open Worldwide Application Security Project | 웹 보안 표준을 정리하는 비영리 단체. "Top 10"이 대표 문서 | [04-deployment 06](./04-deployment/notes/06-security.md) |
| Rate Limiting | — | 일정 시간당 요청 수를 제한해 남용을 막는 것 | [04-deployment 06](./04-deployment/notes/06-security.md) |
| 모노레포 (Monorepo) | — | 여러 패키지(frontend/backend/shared)를 한 저장소에서 관리하는 방식 | [05-capstone monorepo](./05-capstone/docs/monorepo.md) |
| ERD | Entity-Relationship Diagram | 테이블과 그 관계를 그린 그림. DB 설계의 산출물 | [05-capstone](./05-capstone/README.md) |
| MVP | Minimum Viable Product | "이것만 되면 내놓는다"의 최소 기능 범위 | [05-capstone](./05-capstone/docs/requirements.md) |
