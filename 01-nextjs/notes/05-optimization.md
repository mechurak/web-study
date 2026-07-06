# 05. 최적화 — 이미지/폰트, 메타데이터(SEO)

> **핵심 질문**
> - `<img>` 대신 `next/image`를 쓰면 구체적으로 무엇이 최적화되는가?
> - SEO에서 메타데이터가 왜 중요한가? Next.js는 어떻게 페이지별 메타데이터를 만드는가?
> - Core Web Vitals(LCP, CLS, INP)가 무엇이고 Next.js 기능들이 각각 어디에 기여하는가?

## 개념 정리

### 이미지
- [ ] `next/image` — 자동 리사이징, WebP/AVIF 변환, lazy loading
- [ ] `fill`, `sizes`, `priority` 옵션의 의미
- [ ] 외부 이미지 도메인 허용 (`remotePatterns`)

### 폰트
- [ ] `next/font` — 셀프 호스팅, layout shift 방지

### 메타데이터 & SEO
- [ ] `metadata` export / `generateMetadata` (동적 페이지)
- [ ] Open Graph 이미지, `sitemap.ts`, `robots.ts`

### 번들 & 성능
- [ ] `dynamic()` 지연 로딩
- [ ] 번들 분석 (`@next/bundle-analyzer`)
- [ ] Lighthouse / Vercel Speed Insights로 측정

## 실습 (practice 앱에서)

- [ ] 이미지 갤러리 페이지 — `<img>` 버전과 `next/image` 버전의 네트워크 탭 비교
- [ ] 상세 페이지에 `generateMetadata`로 동적 타이틀/OG 태그 적용
- [ ] Lighthouse 점수 측정 → 최적화 → 재측정 기록 남기기

## 참고 자료

- https://nextjs.org/docs/app/building-your-application/optimizing
