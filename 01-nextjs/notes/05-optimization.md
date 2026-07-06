# 05. 최적화 — 이미지/폰트, 메타데이터(SEO)

> **핵심 질문**
> - `<img>` 대신 `next/image`를 쓰면 구체적으로 무엇이 최적화되는가?
> - SEO에서 메타데이터가 왜 중요한가? Next.js는 어떻게 페이지별 메타데이터를 만드는가?
> - Core Web Vitals(LCP, CLS, INP)가 무엇이고 Next.js 기능들이 각각 어디에 기여하는가?

## 개념 정리

### 먼저: Core Web Vitals — 무엇을 최적화하는가

최적화는 "빠르게"라는 막연한 목표가 아니라 Google이 정의한 측정 가능한 3개 지표를 목표로 한다. 검색 랭킹에도 반영되므로 SEO와도 직결된다.

| 지표 | 측정하는 것 | 좋음 기준 | 주로 망치는 원인 |
|---|---|---|---|
| **LCP** (Largest Contentful Paint) | 가장 큰 콘텐츠(보통 히어로 이미지)가 보이기까지 | 2.5초 이내 | 최적화 안 된 큰 이미지, 느린 서버 응답 |
| **CLS** (Cumulative Layout Shift) | 로딩 중 레이아웃이 밀리는 정도 | 0.1 이하 | 크기 명시 없는 이미지, 폰트 교체(FOUT), 늦게 끼어드는 배너 |
| **INP** (Interaction to Next Paint) | 클릭/입력 후 화면 반응까지 | 200ms 이내 | 무거운 JS, 긴 hydration, 메인 스레드 블로킹 |

> INP는 2024년에 FID(First Input Delay)를 대체했다. FID는 "첫 입력의 지연"만 쟀지만 INP는 페이지 생애 전체의 상호작용 반응성을 잰다.

이 챕터의 도구들은 각각 특정 지표를 공략한다: `next/image` → LCP·CLS, `next/font` → CLS, `dynamic()` → INP.

### 이미지 — `next/image`

`<img src="/photo.jpg">`는 원본을 그대로 보낸다. 4000px 원본을 모바일 360px 화면에도 그대로 보내는 것. `next/image`는 이를 요청 시점에 자동으로 처리한다:

- **자동 리사이징**: 요청한 기기 폭에 맞는 크기로 서버(Vercel의 이미지 최적화 서비스)가 변환해서 응답. `srcset`을 자동 생성해 브라우저가 알맞은 크기를 고르게 한다.
- **포맷 변환**: 브라우저가 지원하면 WebP/AVIF로 변환 (JPEG 대비 30~50% 작음).
- **lazy loading 기본값**: 뷰포트 밖 이미지는 스크롤로 가까워질 때 로드.
- **CLS 방지**: `width`/`height`(또는 `fill`)를 강제해서 이미지 로드 전에 자리를 확보한다. 크기를 모르면 빌드 에러 — "귀찮음"이 아니라 CLS 방지 장치다.

```tsx
import Image from 'next/image'

// 크기를 아는 경우
<Image src="/hero.jpg" alt="히어로" width={1200} height={600} priority />

// 부모 요소를 채우는 경우 (크기를 모르는 외부 이미지 등)
<div className="relative h-64">
  <Image src={url} alt="..." fill className="object-cover" sizes="(max-width: 768px) 100vw, 33vw" />
</div>
```

주요 옵션의 의미:

- `priority` — lazy loading을 끄고 preload한다. **뷰포트 상단의 LCP 후보 이미지(히어로)에만** 붙인다. 남발하면 오히려 초기 로드가 무거워진다.
- `fill` — 부모(`position: relative` 필요)를 채운다. `object-cover`와 조합.
- `sizes` — "이 이미지가 화면에서 차지할 폭"을 브라우저에 알려줘서 `srcset` 중 알맞은 것을 고르게 한다. `fill`이나 반응형 그리드에서는 지정하지 않으면 100vw로 가정해 필요 이상 큰 이미지를 받는다.

외부 이미지는 보안상 기본 차단이다. 최적화 서버가 아무 URL이나 프록시하면 악용될 수 있어서, `next.config.ts`의 `images.remotePatterns`에 허용 도메인을 명시해야 한다 (Supabase Storage 붙일 때 필요해진다 → 02-supabase 04장).

### 폰트 — `next/font`

웹폰트의 두 가지 문제: ① Google Fonts CDN 요청 = 외부 네트워크 의존 + 개인정보 이슈, ② 폰트 로드 전후 글자가 바뀌며 레이아웃이 밀림(CLS).

`next/font`는 빌드 시 폰트 파일을 **다운로드해서 내 배포에 포함**(셀프 호스팅)하고, `size-adjust`된 fallback 폰트를 자동 생성해 교체 시 밀림을 최소화한다.

```tsx
// app/layout.tsx
import { Noto_Sans_KR } from 'next/font/google'

const notoSansKr = Noto_Sans_KR({ subsets: ['latin'], display: 'swap' })

export default function RootLayout({ children }) {
  return <html lang="ko" className={notoSansKr.className}><body>{children}</body></html>
}
```

### 메타데이터 & SEO

크롤러와 SNS 미리보기 봇은 페이지의 `<head>`를 읽는다. 제목·설명·OG 태그가 없으면 검색 결과와 공유 카드가 엉망이 된다. App Router에서는 `<head>`를 직접 만지지 않고 **데이터로 선언**한다:

```tsx
// 정적 페이지: 객체 export
export const metadata = { title: '메모장', description: '...' }

// 동적 페이지: 데이터를 fetch해서 생성
export async function generateMetadata({ params }) {
  const memo = await getMemo(params.id)
  return {
    title: memo.title,
    openGraph: { title: memo.title, images: [`/og?id=${memo.id}`] },
  }
}
```

- 레이아웃과 페이지의 metadata는 **병합**된다 (레이아웃에 공통값, 페이지에서 덮어쓰기). `title.template`(`'%s | 메모장'`)으로 접미사 패턴을 만든다.
- `app/sitemap.ts`, `app/robots.ts` — 파일을 만들면 `/sitemap.xml`, `/robots.txt`가 자동 생성된다. 사이트맵은 크롤러에게 "여기 이런 페이지들이 있다"고 알려주는 목록.
- OG 이미지는 `opengraph-image.tsx`로 코드에서 동적 생성할 수도 있다 (`ImageResponse`).

### 번들 & 성능

- **`dynamic()` 지연 로딩** — 초기 화면에 필요 없는 무거운 컴포넌트(차트, 에디터, 지도)를 별도 청크로 분리해 실제 사용 시점에 로드한다. 초기 JS가 줄면 hydration이 빨라져 INP에 기여.
  ```tsx
  const Chart = dynamic(() => import('./Chart'), { ssr: false, loading: () => <Skeleton /> })
  ```
- **번들 분석** — `@next/bundle-analyzer`로 "무엇이 번들을 크게 만드는가"를 시각화. 감으로 최적화하지 말고 측정부터.
- **측정 도구 구분** — Lighthouse(DevTools)는 내 컴퓨터에서의 **실험실 측정**, Vercel Speed Insights는 실제 방문자들의 **필드 측정**(RUM). 실험실 점수가 좋아도 필드가 나쁠 수 있다(느린 기기·네트워크의 실사용자). 04-deployment 05장에서 필드 측정을 붙인다.

## 실습 (practice 앱에서)

- [ ] 이미지 갤러리 페이지 — 같은 이미지들을 `<img>` 버전과 `next/image` 버전으로 만들고 Network 탭 비교 (전송량, 포맷, 로드 시점)
  → **의미**: "자동 최적화"가 실제 몇 KB를 아끼는지 숫자로 확인한다. 00-overview에서 배운 Network 탭 읽기의 연장이고, `srcset`·WebP 변환이 응답에 실제로 나타나는 것을 본다.
- [ ] 상세 페이지에 `generateMetadata`로 동적 타이틀/OG 태그 적용, https://www.opengraph.xyz 같은 미리보기 도구로 확인
  → **의미**: "동적 라우트(02장) + 데이터 페칭(03장) → 메타데이터"로 앞 챕터가 합류하는 지점. 카톡/슬랙에 링크 공유했을 때 카드가 뜨는 원리를 직접 만든다.
- [ ] Lighthouse 점수 측정 → `priority`/`sizes`/`dynamic()` 적용 → 재측정. 전후 점수와 무엇이 몇 점 올렸는지 이 노트 하단에 기록
  → **의미**: 최적화는 측정-개선-재측정의 루프임을 체화한다. 여기서 잰 실험실 점수는 04-deployment 05장에서 실사용자 필드 데이터(Speed Insights)와 비교하게 된다.

## 참고 자료

- https://nextjs.org/docs/app/building-your-application/optimizing
- https://web.dev/articles/vitals — Core Web Vitals 공식 설명

## 측정 기록

(실습 3번 하면서 채우기: 날짜 / 페이지 / 변경 전후 점수 / 적용한 것)
