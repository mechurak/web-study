# web-study

Next.js + Vercel + Supabase + NestJS로 웹 서비스 개발~배포 전 과정을 학습하는 레포.

## 학습 로드맵 (폴더 번호 = 추천 순서)

| 챕터 | 주제 | 역할 | 상태 |
|---|---|---|---|
| [00-overview](./00-overview/) | 웹 서비스 아키텍처 큰 그림 | 전체 조망 | ⬜ |
| [01-nextjs](./01-nextjs/) | Next.js | Frontend (+ 가벼운 백엔드) | ⬜ |
| [02-supabase](./02-supabase/) | Supabase | DB + 인증 + 스토리지 (BaaS) | ⬜ |
| [03-nestjs](./03-nestjs/) | NestJS | 본격 Backend API 서버 | ⬜ |
| [04-deployment](./04-deployment/) | Vercel & 배포 전반 | 배포 + 운영 | ⬜ |
| [05-capstone](./05-capstone/) | 통합 프로젝트 | 전 레이어 관통 실전 | ⬜ |

## 폴더 구성 규칙

각 챕터 폴더는 같은 구조를 따른다:

- `README.md` — **왜 이 기술을 쓰는가** + 챕터 목차
- `notes/` — 주제별 개념 정리 (학습하며 채워나감)
- `practice/` — 실행 가능한 실습 코드 (기술당 프로젝트 1개, 챕터별로 확장)

## 학습 흐름의 의도

1. Next.js로 화면과 간단한 API를 먼저 만들고,
2. Supabase로 데이터·인증을 붙인 뒤,
3. "Next API Route만으로 부족한 순간"을 체감하고 NestJS로 넘어간다.
4. 배포는 각 실습 앱을 실제로 올려보며 정리하고,
5. 마지막에 작은 서비스 하나를 처음부터 끝까지 만든다.
