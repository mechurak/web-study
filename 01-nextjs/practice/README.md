# Next.js 실습 앱

이 폴더에 실습용 Next.js 앱 **하나**를 만들고, 각 노트 챕터의 실습을 페이지 단위로 추가해나간다.

## 시작하기

```bash
cd 01-nextjs/practice
npx create-next-app@latest app --typescript --app --eslint
cd app
npm run dev
```

## 챕터별 실습 진행 상황

- [ ] 01 라우팅: 홈/목록/상세 페이지 + 레이아웃
- [ ] 02 렌더링: SSG/SSR/CSR 비교 페이지
- [ ] 03 데이터 페칭: 외부 API 목록 + Server Action 메모 폼
- [ ] 04 API Routes: 메모 CRUD API + 미들웨어
- [ ] 05 최적화: 이미지 갤러리 + 동적 메타데이터
- [ ] 06 Tailwind: 반응형 카드 그리드 + 다크 모드 토글
- [ ] 07 shadcn/ui: 메모 UI 컴포넌트 교체 + Form/Dialog/Toast

> 이 앱은 02-supabase 챕터에서 Supabase를 붙이는 베이스로 계속 사용한다.
