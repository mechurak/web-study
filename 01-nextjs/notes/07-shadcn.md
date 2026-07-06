# 07. shadcn/ui — 복사해서 소유하는 컴포넌트

> **핵심 질문**
> - shadcn/ui는 왜 "라이브러리가 아니다"라고 하는가? npm 설치(MUI, Ant Design)와 코드 복사 방식의 트레이드오프는?
> - Radix UI가 밑에서 해주는 것은? (접근성, 키보드 인터랙션, 포커스 관리)
> - 디자인 토큰이 CSS 변수로 되어 있어서 얻는 것은? (테마 교체, 다크 모드)

## 개념 정리

### 동작 방식 — "설치"가 아니라 "복사"

MUI는 `npm install` 하면 컴포넌트가 `node_modules`에 들어가고, 나는 그것을 import해서 **쓴다**. shadcn은 다르다:

```bash
npx shadcn@latest init          # 설정 + 토큰(CSS 변수) + cn() 헬퍼 셋업
npx shadcn@latest add button    # button.tsx 소스가 components/ui/에 "복사"됨
```

컴포넌트의 **소스 코드가 내 프로젝트 파일이 된다**. 그래서 "컴포넌트 라이브러리가 아니라 코드 배포 시스템"이라고 자칭한다. 트레이드오프:

| | npm 라이브러리 (MUI 등) | shadcn (코드 복사) |
|---|---|---|
| 커스터마이징 | 라이브러리가 열어준 prop/테마 API 안에서만 | 소스를 직접 고침 — 한계 없음 |
| 업데이트 | `npm update` 한 방 | 수동 (diff 보고 직접 반영) |
| 스타일 싸움 | 라이브러리 스타일을 덮어쓰는 CSS 전쟁 | 없음 — 애초에 내 Tailwind 코드 |
| 번들 | 라이브러리 전체 의존 | 쓰는 컴포넌트 파일만 존재 |

이 방식이 성립하는 배경: 디자인이 조금이라도 독자적인 서비스는 결국 라이브러리 스타일을 덮어쓰는 데 더 많은 시간을 쓴다는 경험칙. 처음부터 소스를 주고 "고쳐 쓰라"는 것.

### 3층 구조 — 각 층이 하는 일

shadcn 컴포넌트 하나를 열어보면 세 가지 조합이다:

1. **Radix UI** (동작·접근성) — headless 컴포넌트. 스타일은 없고 **동작만** 있다: Dialog가 열리면 포커스를 가두고(focus trap), ESC로 닫히고, 배경 스크롤을 막고, 스크린리더용 `aria-*` 속성을 관리한다. 이런 접근성 로직은 직접 만들면 어렵고 빠뜨리기 쉬운 부분 — "밑에서 해주는 것"의 정체다.
2. **Tailwind** (스타일) — 06장에서 배운 유틸리티로 겉모습을 입힌다. 색은 하드코딩이 아니라 CSS 변수 토큰(`bg-primary`)을 쓴다.
3. **CVA** (class-variance-authority, variant 관리) — "variant별 클래스 조합"을 선언적으로:

```tsx
const buttonVariants = cva('inline-flex items-center rounded-md ...', {
  variants: {
    variant: {
      default: 'bg-primary text-primary-foreground hover:bg-primary/90',
      destructive: 'bg-destructive text-white hover:bg-destructive/90',
      outline: 'border bg-background hover:bg-accent',
    },
    size: { default: 'h-9 px-4', sm: 'h-8 px-3', lg: 'h-10 px-6' },
  },
  defaultVariants: { variant: 'default', size: 'default' },
})

<Button variant="destructive" size="sm">삭제</Button>
```

06장의 `cn()`이 여기서 합류한다 — `className` prop으로 받은 것을 `cn(buttonVariants({ variant, size }), className)`으로 병합해서, 사용처가 언제든 스타일을 덮어쓸 수 있다.

### 핵심 컴포넌트 — 우선 익힐 것

- **Button, Input, Card** — 기본 재료. Card는 `Card/CardHeader/CardTitle/CardContent`처럼 조립식(composition) API.
- **Dialog, DropdownMenu** — Radix의 가치가 보이는 것들. 키보드(Tab/ESC/화살표)로만 조작해볼 것.
- **Form** — react-hook-form + zod를 묶는 표준 패턴:

```tsx
const schema = z.object({ title: z.string().min(1, '제목을 입력하세요') })
const form = useForm({ resolver: zodResolver(schema) })

<Form {...form}>
  <FormField control={form.control} name="title" render={({ field }) => (
    <FormItem>
      <FormLabel>제목</FormLabel>
      <FormControl><Input {...field} /></FormControl>
      <FormMessage />   {/* zod 에러 메시지가 자동으로 여기 뜬다 */}
    </FormItem>
  )} />
</Form>
```

스키마(zod)가 검증 규칙과 에러 메시지의 단일 원천이 된다 — 이 스키마를 서버 검증과 공유하는 것이 09장의 주제.

- **Toast** — 현 세대 shadcn은 자체 toast 대신 **sonner**를 권장한다 (`npx shadcn@latest add sonner`, `toast('저장됨')`).

### 테마 — CSS 변수가 단일 원천

`init`이 `globals.css`에 시맨틱 토큰을 깔아준다:

```css
:root { --background: ...; --primary: ...; --destructive: ...; }
.dark  { --background: ...; --primary: ...; }   /* 다크 모드는 변수 재정의 */
```

컴포넌트는 `bg-primary`처럼 **역할 이름**만 참조하므로:

- **테마 교체 = 변수 값만 교체**. 컴포넌트 코드는 한 줄도 안 바뀐다. https://ui.shadcn.com/themes 에서 색조합을 골라 붙여넣으면 끝.
- **다크 모드 = `.dark`에서 변수 재정의**. `dark:` 클래스를 컴포넌트마다 다는 게 아니라 토큰 층에서 한 번에 해결된다(06장에서 손으로 `dark:`를 달아본 것과 비교해볼 것).
- 전환 토글은 `next-themes`가 담당 — `<html>`에 `.dark` 클래스를 붙이고 떼며, OS 설정 연동과 로컬 저장, 첫 페인트 깜빡임(FOUC) 방지까지 처리한다.

### 서버 컴포넌트 호환

Button, Card처럼 상태 없는 것은 서버 컴포넌트에서 그대로 쓸 수 있다. Dialog, DropdownMenu, Form처럼 상태·이벤트가 있는 것은 소스에 `"use client"`가 이미 선언돼 있다 — 02장의 경계 설계("잎만 클라이언트")가 컴포넌트 단위로 실현된 형태. 소스가 내 프로젝트에 있으니 파일을 열어 직접 확인할 수 있다는 게 학습 관점의 덤이다.

### 판단 기준 — shadcn vs 완제품 라이브러리

- **shadcn이 맞는 경우**: 서비스 고유의 디자인으로 갈 것이고, Tailwind를 쓰고 있고, 컴포넌트를 직접 소유·수정할 의지가 있을 때. (이 레포의 선택)
- **MUI/Ant가 맞는 경우**: 디자인 차별화가 필요 없는 내부 도구·어드민을 최고 속도로 찍어낼 때, DataGrid 같은 초고기능 컴포넌트가 당장 필요할 때.
- 기준 한 줄: **"이 UI의 디자인이 자산인가, 비용인가"** — 자산이면 소유(shadcn), 비용이면 구매(완제품).

## 실습 (practice 앱에서)

- [ ] `npx shadcn@latest init` 후 `components/ui/button.tsx` 소스를 열어 CVA 구조 읽기, 메모 UI를 Card/Button/Input으로 교체
  → **의미**: "복사해서 소유"를 체감하는 첫 단계. 06장에서 만든 `cn()`과 카드 그리드가 shadcn 코드와 그대로 연결되는 것을 확인한다.
- [ ] 메모 삭제에 Dialog(확인 모달) 적용 — 마우스 없이 키보드(Tab/ESC/Enter)로만 조작해보기, 완료 알림은 sonner Toast로
  → **의미**: Radix가 주는 접근성(포커스 트랩, 키보드 인터랙션)을 직접 검증. "직접 만든 모달"과의 차이가 여기서 드러난다.
- [ ] 메모 작성 폼을 Form + zod 검증으로 재구현 (03장에서 만든 Server Action에 연결)
  → **의미**: "클라이언트 검증(즉각 피드백) + 서버 처리(Server Action)"의 표준 조합 완성. 이 zod 스키마는 09장에서 서버 검증과 공유되며, capstone까지 계속 쓰는 패턴이다.
- [ ] `next-themes`로 다크 모드 완성 (06장 토글을 시스템 연동 + 저장으로 업그레이드), https://ui.shadcn.com/themes 에서 테마를 골라 변수만 바꿔 전체 색 교체
  → **의미**: "토큰 층에서 테마를 해결한다"의 검증 — 컴포넌트 코드를 안 건드리고 앱 전체의 룩이 바뀌는 것을 확인하면 CSS 변수 설계의 이유가 납득된다.

## 참고 자료

- https://ui.shadcn.com/docs — 공식 문서 (컴포넌트별 사용법)
- https://www.radix-ui.com/primitives — Radix 원본 문서 (동작·접근성 명세)
- https://cva.style/docs — CVA
