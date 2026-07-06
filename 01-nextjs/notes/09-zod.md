# 09. zod — 스키마로 검증하고 타입까지 얻기

> **핵심 질문**
> - TypeScript 타입은 런타임에 사라진다 — 그래서 무엇이 문제인가? (외부에서 들어오는 데이터는 못 믿는다)
> - "스키마 하나로 검증 + 타입 추론"이 왜 강력한가? (`z.infer`)
> - 검증은 어디에서 해야 하는가? — 경계(boundary)마다: 폼 입력, API 요청/응답, 환경변수

## 개념 정리

### 왜 필요한가 — 타입은 컴파일 타임에만 존재한다

`const data: Memo = await res.json()`이라고 써도 TypeScript는 **아무것도 확인하지 않는다**. 타입은 빌드 시 검사 후 지워지는 주석일 뿐이고, `res.json()`의 실제 반환은 서버가 보낸 그 무엇이다. 서버가 필드명을 바꿨거나, 유저가 폼을 우회해 조작된 요청을 보냈어도 타입 시스템은 침묵한다 — 버그는 한참 뒤 엉뚱한 곳에서 터진다.

결론: **내 코드 밖에서 들어오는 모든 데이터는 런타임에 검증해야 한다.** zod는 그 검증 규칙(스키마)을 한 번 정의하면 **런타임 검증과 컴파일 타임 타입을 동시에** 주는 도구다.

### 기본기

```ts
import { z } from 'zod'

const memoSchema = z.object({
  id: z.string().uuid(),
  title: z.string().min(1, '제목은 필수').max(100),
  content: z.string().default(''),
  isFavorite: z.boolean(),
  createdAt: z.coerce.date(),        // 문자열로 와도 Date로 변환
})

type Memo = z.infer<typeof memoSchema>   // 스키마에서 타입을 뽑는다
```

- **`z.infer`가 핵심**: 타입을 interface로 또 쓰지 않는다. 스키마가 유일한 정의이고 타입은 파생물 — 규칙과 타입이 어긋날 수가 없다.
- **`parse` vs `safeParse`**:
  - `schema.parse(data)` — 실패 시 `ZodError`를 **throw**. "여기서 실패하면 프로그램이 잘못된 것"인 곳(환경변수, 내부 불변조건)에 적합.
  - `schema.safeParse(data)` — `{ success: true, data } | { success: false, error }`를 **반환**. 실패가 정상 흐름인 곳(유저 입력)에 적합. 폼 에러 표시로 이어진다.
- **transform / refine** — 검증을 넘어 변환과 커스텀 규칙:

```ts
const signupSchema = z.object({
  email: z.string().email().transform((v) => v.toLowerCase()),
  password: z.string().min(8),
  confirm: z.string(),
}).refine((v) => v.password === v.confirm, {
  message: '비밀번호가 일치하지 않습니다',
  path: ['confirm'],   // 에러를 어느 필드에 붙일지
})
```

### 경계마다 검증하기 (이 스택에서의 사용처 지도)

원칙: **신뢰 경계(trust boundary)를 데이터가 넘을 때마다 검증한다.** 내부 함수 사이에서는 타입으로 충분하다.

| 경계 | 무엇이 들어오나 | 어떻게 |
|---|---|---|
| **폼** | 유저 입력 | react-hook-form + `zodResolver` — [07-shadcn](./07-shadcn.md)의 Form 패턴. 필드별 에러 메시지가 자동으로 연결된다 |
| **Server Action** | FormData (조작 가능!) | 액션 첫 줄에서 `safeParse`. **Server Action도 공개 HTTP 엔드포인트다** — 클라이언트 검증은 UX용이고, 서버 검증이 보안이다 |
| **Route Handler / 외부 API 응답** | JSON | `await res.json()`의 반환은 사실상 `any`다. `schema.parse(json)`을 거쳐야 비로소 믿을 수 있는 타입이 된다 |
| **환경변수** | `process.env` (전부 `string \| undefined`) | 부팅 시 스키마로 검증 — 빠진 변수를 배포 후가 아니라 시작 시점에 잡는다. Next.js에서는 `@t3-oss/env-nextjs`가 이 패턴을 정형화 (03-nestjs 05장의 Nest 버전과 같은 아이디어) |
| **NestJS** | 요청 DTO(Data Transfer Object) | 기본은 class-validator지만 `nestjs-zod`로 zod를 쓸 수도 있다. 트레이드오프: class-validator는 Nest 생태계(Swagger 자동 문서화)와 결합이 깊고, zod는 **프론트와 스키마를 공유**할 수 있다. 모노레포로 갈 거면 zod 통일이 강력해진다 |

Server Action 검증의 전형적 형태:

```ts
'use server'
export async function createMemo(formData: FormData) {
  const parsed = createMemoSchema.safeParse(Object.fromEntries(formData))
  if (!parsed.success) {
    return { error: parsed.error.flatten().fieldErrors }  // 폼에 필드별 에러 반환
  }
  await db.insert(parsed.data)   // 여기부터는 타입도 값도 믿을 수 있다
}
```

### 스키마 공유 — 프론트와 백엔드가 같은 진실을 본다

**파생 스키마**: Create/Update용 타입을 따로 안 만들고 원본에서 뽑는다. 원본이 바뀌면 파생도 따라온다.

```ts
const createMemoSchema = memoSchema.omit({ id: true, createdAt: true })
const updateMemoSchema = createMemoSchema.partial()   // 전 필드 optional
type CreateMemoInput = z.infer<typeof createMemoSchema>
```

**같은 스키마를 양쪽에서**: 프론트 폼 검증(07장)과 백엔드 API 검증(Server Action 또는 NestJS)이 **동일한 스키마 객체**를 import하면, "프론트는 100자까지 허용하는데 백엔드는 50자에서 거절" 같은 불일치가 원천적으로 사라진다. 이것이 zod가 이 스택의 접착제인 이유.

다만 Next 앱과 Nest 서버가 다른 패키지라서 "같은 파일을 import"하려면 공유 패키지(`packages/shared`)가 필요하다 → 모노레포 구성: [05-capstone/docs/monorepo.md](../../05-capstone/docs/monorepo.md)

> zod 버전 참고: 2024년 이후 v3 → v4로 넘어가며 일부 API가 바뀌었다(`.email()` 등 문자열 검증 표기, 에러 커스터마이징 방식 등). 이 노트의 `z.object`/`z.infer`/`parse`/`safeParse`/`omit`/`partial`은 안정 API지만, 세부 문법은 설치한 버전의 공식 문서(https://zod.dev)로 확인할 것.

## 실습 (practice 앱에서)

- [ ] 메모 스키마(`memoSchema`) 정의, Create/Update를 `.omit()`/`.partial()` 파생으로 생성
  → **의미**: "타입을 두 번 쓰지 않는다"를 구조로 체험. 여기서 만든 스키마가 이후 실습 전부의 원본이 되고, capstone에서 `packages/shared`로 이사 가는 바로 그 파일이다.
- [ ] 07장 shadcn 폼 + Server Action 양쪽에서 같은 스키마로 검증 — 폼으로는 통과 못 하는 값을 curl로 Server Action 엔드포인트에 직접 보내 서버에서도 거절되는 것 확인
  → **의미**: "클라이언트 검증은 UX, 서버 검증이 보안"을 직접 증명하는 실습. 04-deployment 06장(보안)의 입력 검증 항목과 03-nestjs 02장(DTO 검증)이 전부 이 원칙의 변주다.
- [ ] 외부 API 응답을 zod로 파싱하되, 스키마에 일부러 필드 오타를 내서 런타임 에러가 **어디서** 나는지 확인 — zod 없이 오타 냈을 때 어디까지 조용히 흘러가는지와 비교
  → **의미**: "타입은 런타임에 사라진다"는 문장을 버그의 발생 위치 차이로 체감한다. 검증이 있으면 경계에서 즉시 터지고(원인 명확), 없으면 화면 어딘가에서 undefined로 터진다(원인 추적 비용).

## 참고 자료

- https://zod.dev
- https://env.t3.gg — 환경변수 검증 패턴 (`@t3-oss/env-nextjs`)
