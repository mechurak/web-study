# 10. 프론트엔드 테스트 — Playwright E2E, React Testing Library

> **핵심 질문**
> - 백엔드 테스트(03-nestjs 05장)와 달리 프론트 테스트는 무엇을 검증하는가? — "유저가 보는 동작"
> - E2E(End-to-End)와 컴포넌트 테스트의 비용/신뢰도 트레이드오프는? (테스팅 피라미드 vs 트로피)
> - "구현이 아니라 동작을 테스트하라"는 말의 의미는? (클래스명이 아니라 role/텍스트로 찾기)

## 개념 정리

### 테스트 층위 정하기

백엔드 테스트는 "함수에 X를 넣으면 Y가 나온다"를 검증하지만, 프론트 테스트의 대상은 **유저가 보고 겪는 동작**이다 — "버튼을 누르면 메모가 목록에 나타난다". 그래서 층위 구분도 "얼마나 실제 유저 경험에 가깝게 돌리느냐"로 나뉜다:

| 층위 | 도구 | 검증하는 것 | 비용 | 신뢰도 |
|---|---|---|---|---|
| **E2E** | Playwright | 실제 브라우저 + 실제 서버 + 실제 DB로 핵심 시나리오 전체 | 느리고 깨지기 쉬움 | 최고 — "진짜 되는가" |
| **컴포넌트** | RTL(React Testing Library) + Vitest | 컴포넌트 하나를 가짜 DOM(jsdom)에서 렌더링해 상호작용 | 빠름 | 중간 — 조립은 검증 못 함 |

전통적 "테스팅 피라미드"(유닛 잔뜩, E2E 조금)에 대해, 프론트엔드에서는 **테스팅 트로피**(Kent C. Dodds)가 더 통용된다 — 유닛보다 **통합 수준(여러 컴포넌트가 함께 동작)에 가장 많은 투자**를 하라는 것. 이유: 프론트 버그는 대부분 함수 하나가 아니라 조립(연결, 상태 전달)에서 나기 때문이다.

**무엇을 테스트 안 할지**도 정한다: 로직 없는 표시용 컴포넌트(스타일 확인은 눈으로), 스냅샷 테스트 남발(뭐가 왜 바뀌었는지 안 알려주고 아무나 업데이트 누르게 됨), 서드파티 라이브러리 자체의 동작. 원칙: **깨졌을 때 "유저에게 뭐가 문제인지" 말해주는 테스트만 남긴다.**

### "구현이 아니라 동작을 테스트하라"

```ts
// ❌ 구현 세부에 결합 — 리팩터링(클래스명 변경)만 해도 깨짐
page.locator('.memo-form > button.submit-btn')

// ✅ 유저가 인식하는 방식으로 — 스크린리더가 보는 것과 동일
page.getByRole('button', { name: '저장' })
```

`getByRole`은 접근성 트리(role + accessible name)로 요소를 찾는다. 이 방식의 이중 효과: 테스트가 리팩터링에 강해지고, **role로 찾을 수 없는 UI는 접근성이 나쁜 UI라는 신호**가 된다. 테스트가 접근성 검사를 겸하는 셈.

### Playwright

**기본 형태**:

```ts
import { test, expect } from '@playwright/test'

test('메모를 작성하면 목록에 나타난다', async ({ page }) => {
  await page.goto('/memos')
  await page.getByLabel('제목').fill('장보기')
  await page.getByRole('button', { name: '저장' }).click()
  await expect(page.getByRole('listitem').filter({ hasText: '장보기' })).toBeVisible()
})
```

- `expect(locator)`의 단언은 **자동 대기(auto-wait)** 한다 — 요소가 나타날 때까지 재시도하므로 `sleep`을 직접 넣지 않는다. 이것이 Selenium 세대 도구 대비 핵심 개선.
- `npx playwright test --ui` — UI 모드로 각 스텝을 눈으로 보며 디버깅.

**인증 상태 재사용 (`storageState`)**: 모든 테스트가 로그인부터 시작하면 느리고, 로그인 화면이 바뀌면 전부 깨진다. **setup 프로젝트에서 한 번 로그인해 쿠키/스토리지를 파일로 저장**하고, 나머지 테스트는 그 파일을 물고 로그인된 상태로 시작한다. 설정법: https://playwright.dev/docs/auth

**테스트 데이터 전략**: E2E는 진짜 DB를 건드린다. 프로덕션 Supabase가 아니라 **로컬 Supabase(`supabase start`)나 테스트 전용 프로젝트**를 쓰고, 테스트 전용 유저 계정 + 테스트가 만든 데이터는 테스트가 정리(또는 매번 DB 리셋)하는 규칙을 세운다. "어제는 통과했는데 오늘은 실패"의 주범이 남은 데이터다.

**실패 디버깅 — trace viewer**: CI에서 실패한 테스트의 전 과정(스크린샷 타임라인, 네트워크, 콘솔)을 녹화한 trace 파일을 로컬에서 재생할 수 있다. `trace: 'on-first-retry'` 설정이 관례.

### React Testing Library (+ Vitest)

RTL은 컴포넌트를 jsdom에 렌더링하고 **유저 관점 쿼리**로 검증한다. 테스트 러너로는 Jest 대신 Vitest가 요즘 기본값(빠르고 ESM/TS가 매끄러움).

```tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

test('빈 제목으로 제출하면 에러가 보인다', async () => {
  const user = userEvent.setup()
  render(<MemoForm />)
  await user.click(screen.getByRole('button', { name: '저장' }))
  expect(await screen.findByText('제목은 필수')).toBeInTheDocument()
})
```

- **쿼리 우선순위**: `getByRole` > `getByLabelText` > `getByPlaceholderText` > `getByText` > … > `getByTestId`(최후 수단). 위일수록 유저가 실제로 인식하는 방식이다.
- **`userEvent`를 쓸 것** (`fireEvent`보다) — 실제 유저처럼 포커스 → 키 입력 → 이벤트 순서를 재현한다.
- **비동기는 `findBy*`** — 상태 업데이트 후 나타나는 요소를 기다린다 (`getBy`는 즉시 실패).
- **한계**: `async` 서버 컴포넌트는 jsdom에서 렌더링할 수 없다 — RTL 대상은 **클라이언트 컴포넌트와 훅**이고, 서버 컴포넌트가 낀 흐름은 E2E가 담당한다. 이 한계가 "폼·버튼 같은 상호작용 단위는 RTL, 페이지 전체 흐름은 Playwright"라는 분업의 근거다.

### CI 연결

- GitHub Actions에서 PR마다 실행: lint + typecheck + Vitest(빠름, 항상) + Playwright(핵심 시나리오만). 04-deployment 04장의 CI(Continuous Integration) 파이프라인에 편입되고, 브랜치 보호로 "깨지면 머지 불가"가 된다.
- 한 단계 위: **Vercel 프리뷰 배포 URL을 대상으로 E2E 실행** — 내 컴퓨터의 dev 서버가 아니라 실제 배포 환경(진짜 환경변수, 진짜 serverless)을 검증하므로 "로컬에선 됐는데" 부류의 문제를 잡는다.

## 실습 (practice 앱에서)

- [ ] Playwright 셋업(`npm init playwright@latest`), 핵심 시나리오 E2E 작성: 로그인 → 메모 작성 → 목록에 보임 → 삭제
  → **의미**: 이 테스트 하나가 스택 전체(Next 렌더링, Server Action, Supabase 인증·DB·RLS)를 관통하는 통합 검증이다. 앞 챕터들에서 만든 모든 것이 "실제로 함께 동작하는가"의 답.
- [ ] 인증 storageState 재사용 설정 — setup에서 1회 로그인, 이후 테스트는 로그인 생략
  → **의미**: E2E를 유지 가능한 속도로 만드는 필수 패턴. Supabase 세션이 쿠키에 어떻게 담기는지(02-supabase 05장)를 테스트 관점에서 다시 보게 된다.
- [ ] 메모 폼 컴포넌트를 RTL + Vitest로 테스트 — 빈 제목 제출 시 zod 검증 에러(09장)가 화면에 표시되는 것 확인
  → **의미**: "폼 검증"이라는 한 기능을 RTL(컴포넌트 단위, 빠름)과 E2E(흐름 단위, 느림) 어느 층위에서 검증할지 감을 잡는다. `getByRole`로 못 찾는 요소가 있다면 접근성 개선 신호.
- [ ] CI에서 Vitest + Playwright가 돌고, 일부러 테스트를 깨뜨린 PR이 머지 차단되는 것까지 확인
  → **의미**: 테스트의 최종 목적지는 "사람이 돌리는 것"이 아니라 "머지 관문"이다. 04-deployment 04장의 CI/브랜치 보호와 만나 배포 파이프라인이 완성된다.

## 참고 자료

- https://playwright.dev/docs/intro
- https://testing-library.com/docs/react-testing-library/intro
- https://kentcdodds.com/blog/write-tests — "Write tests. Not too many. Mostly integration."
