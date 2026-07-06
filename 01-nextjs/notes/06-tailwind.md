# 06. Tailwind CSS — 유틸리티 퍼스트 스타일링

> **핵심 질문**
> - 유틸리티 퍼스트란? CSS 파일/CSS-in-JS 방식과 비교해 무엇이 좋고 무엇이 불편한가?
> - "클래스가 지저분해진다"는 비판에 대한 답은? (컴포넌트로 추상화, 디자인 시스템)
> - Tailwind는 어떻게 빌드 시 사용한 클래스만 남기는가? (JIT, content 스캔)

## 개념 정리

### 유틸리티 퍼스트 — 무엇이고 왜인가

CSS 클래스에 "의미"(`.memo-card`)를 붙이고 별도 파일에 스타일을 쓰는 대신, **한 가지 스타일 속성 = 한 클래스**인 유틸리티를 마크업에 직접 조합한다:

```tsx
// 전통 방식: 이름 짓고, 파일 오가고, 안 쓰는 CSS가 쌓임
<div className="memo-card">...</div>   // + memo-card를 정의한 .css 파일

// 유틸리티 퍼스트: 마크업만 보면 스타일이 다 보임
<div className="rounded-lg border p-4 shadow-sm hover:shadow-md">...</div>
```

무엇이 좋아지는가:

- **이름 짓기가 사라진다** — `.memo-card-inner-wrapper` 같은 고민 소멸.
- **지역성** — 이 요소의 스타일이 이 줄에 다 있다. 다른 파일의 CSS가 몰래 영향을 주는 일이 없어서, 지워도 안전하다(전통 CSS의 최대 문제는 "이 규칙 지워도 되나?"를 알 수 없다는 것).
- **디자인 토큰이 기본 내장** — `p-4`, `text-sm`은 임의 값이 아니라 정해진 스케일이라 화면 전체의 간격·크기가 자연스럽게 일관된다.
- **CSS-in-JS 대비** — 런타임 비용이 없다(빌드 시 순수 CSS 생성). styled-components류는 서버 컴포넌트와 궁합도 나쁘다(런타임 스타일 주입이 필요해서 대부분 `"use client"` 강제).

불편한 점과 그 답: "클래스가 지저분하다"는 비판은 사실이지만, **반복되는 클래스 조합의 추상화 단위는 CSS 클래스가 아니라 React 컴포넌트**라는 게 Tailwind의 관점이다. `memo-card` 클래스를 만드는 대신 `<MemoCard>` 컴포넌트를 만들면 재사용과 이름 짓기가 이미 해결돼 있다. 07장의 shadcn/ui가 정확히 이 철학의 산물이다.

### 어떻게 사용한 클래스만 남는가 — JIT

Tailwind는 가능한 모든 유틸리티(수백만 개 조합)를 미리 만들어두지 않는다. 빌드 시 **소스 파일을 텍스트 스캔해서 발견된 클래스만 생성**한다(JIT, Just-in-Time). 그래서 최종 CSS가 보통 10KB대로 작다.

함정 하나: 스캐너는 문자열을 그대로 찾으므로 **클래스명을 동적으로 조립하면 안 된다**:

```tsx
<div className={`text-${color}-500`} />          // ❌ 스캐너가 못 찾음 → 스타일 안 나옴
<div className={color === 'red' ? 'text-red-500' : 'text-blue-500'} />  // ✅ 완전한 이름으로
```

### v4 설정 — CSS-first (v3와 다르다!)

Tailwind v4(2025~, `create-next-app` 기본)는 설정 방식이 바뀌었다. v3까지는 `tailwind.config.js`에 JS로 설정했지만, **v4는 CSS 파일 안에서 설정한다**:

```css
/* app/globals.css */
@import "tailwindcss";

@theme {
  --color-brand: oklch(0.7 0.15 250);   /* → bg-brand, text-brand 클래스가 생김 */
  --font-sans: "Noto Sans KR", sans-serif;
}
```

- `@theme`에 선언한 CSS 변수가 곧 디자인 토큰이고, 그대로 유틸리티 클래스가 된다.
- content 경로 지정도 대부분 불필요(자동 감지).
- 검색하다 `tailwind.config.js` + `module.exports` 자료가 나오면 v3 자료다 — 개념은 같지만 문법이 다르니 공식 v4 문서 기준으로 볼 것.

### 자주 쓰는 어휘 (전부 외우지 말고 패턴만)

| 분류 | 예 | 패턴 |
|---|---|---|
| spacing | `p-4` `px-2` `mt-8` `gap-4` `mx-auto` | 방향(x/y/t/b/l/r) + 스케일(1=0.25rem) |
| 크기 | `w-full` `max-w-md` `h-screen` `size-8` | |
| 타이포 | `text-sm` `font-medium` `text-slate-500` | `text-`는 크기와 색 둘 다 |
| 레이아웃 | `flex items-center justify-between` / `grid grid-cols-3` | Flexbox/Grid CSS를 알면 1:1 대응 |
| 장식 | `rounded-lg` `border` `shadow-sm` | |

상태·조건은 **variant 접두사**로: `hover:bg-slate-100`, `focus-visible:ring-2`, `disabled:opacity-50`. `group`을 부모에 두면 `group-hover:visible`로 "부모에 호버하면 자식 스타일 변경"도 된다.

### 반응형 & 다크 모드

- **모바일 퍼스트**: 접두사 없는 클래스가 모바일, `md:` `lg:`는 "그 폭 **이상**에서". `grid-cols-1 md:grid-cols-2 lg:grid-cols-3` = 기본 1열, 768px부터 2열, 1024px부터 3열.
- **다크 모드**: `dark:bg-slate-900`. `dark:` variant가 언제 켜지는지는 전략 선택 — 기본은 OS 설정(`prefers-color-scheme`)을 따르고, 유저 토글을 제공하려면 `<html>`에 클래스를 붙이는 방식으로 설정을 바꾼다(v4에서는 `@custom-variant`로 선언). 토글 구현은 07장에서 `next-themes`로 완성한다.

### 유지보수 패턴

- **`@apply`는 절제** — 유틸리티를 CSS 클래스로 되묶는 기능인데, 남용하면 "이름 짓기 문제"로 회귀한다. 컴포넌트로 추상화가 원칙이고, `@apply`는 마크업을 못 만지는 곳(외부 라이브러리 스타일링 등)에만.
- **클래스 정렬**: `prettier-plugin-tailwindcss` — 클래스 순서를 자동 정렬해서 diff 노이즈를 없앤다.
- **조건부 클래스 — `cn()` 헬퍼**: `clsx`(조건부 조합) + `tailwind-merge`(충돌 해소)의 조합. shadcn이 표준으로 쓴다:

```ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

```tsx
<div className={cn('p-4 bg-white', isSelected && 'bg-blue-50 ring-2')} />
```

`tailwind-merge`가 필요한 이유: `cn('p-4', 'p-2')`처럼 같은 속성이 겹치면 CSS 우선순위는 클래스 순서가 아니라 **생성된 CSS 파일 내 순서**로 정해져 예측 불가다. `twMerge`는 뒤의 것(`p-2`)만 남겨서 "나중에 준 값이 이긴다"는 직관을 보장한다 — 컴포넌트가 `className` prop으로 스타일 덮어쓰기를 받을 때 필수.

## 실습 (practice 앱에서)

- [ ] 기존 페이지들의 스타일을 Tailwind로 정리 (create-next-app에 v4 기본 포함 — `globals.css`의 `@import "tailwindcss"` 확인)
  → **의미**: 유틸리티 어휘를 손에 익히는 기초 반복. `globals.css`를 열어 v4의 CSS-first 설정이 실제로 어떻게 생겼는지 확인한다.
- [ ] 메모 목록을 카드 그리드로 — `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4`, 브라우저 폭을 줄여가며 확인
  → **의미**: 모바일 퍼스트("접두사 없음 = 모바일")를 체감. 이 카드 그리드가 07장에서 shadcn `<Card>`로 교체된다.
- [ ] 다크 모드: 주요 요소에 `dark:` 클래스를 달고, OS 설정을 바꿔 전환 확인
  → **의미**: `dark:` variant의 동작 원리 확인. 07장에서 next-themes로 "유저 토글 + OS 연동"으로 업그레이드하며, 그때 이 클래스들이 그대로 재사용된다.
- [ ] `cn()` 헬퍼를 `lib/utils.ts`에 만들고 선택된 메모 카드 하이라이트에 사용
  → **의미**: 조건부 스타일의 표준 패턴 확보. 07장 shadcn init이 만드는 것과 같은 파일이라, 미리 만들어보면 shadcn 컴포넌트 코드가 바로 읽힌다.
- [ ] (관찰) 빌드 후 생성된 CSS 크기 확인, 동적 클래스 조립(`text-${color}-500`)을 일부러 시도해서 스타일이 안 나오는 것 확인
  → **의미**: JIT의 "소스를 텍스트로 스캔한다"는 동작을 검증. 이 함정은 실무에서 반드시 한 번 밟는 것이라 미리 밟아둔다.

## 참고 자료

- https://tailwindcss.com/docs — v4 기준 공식 문서
- https://tailwindcss.com/blog/tailwindcss-v4 — v4 변경점 (v3 자료와 구분할 때)
