# 07. shadcn/ui — 복사해서 소유하는 컴포넌트

> **핵심 질문**
> - shadcn/ui는 왜 "라이브러리가 아니다"라고 하는가? npm 설치(MUI, Ant Design)와 코드 복사 방식의 트레이드오프는?
> - Radix UI가 밑에서 해주는 것은? (접근성, 키보드 인터랙션, 포커스 관리)
> - 디자인 토큰이 CSS 변수로 되어 있어서 얻는 것은? (테마 교체, 다크 모드)

## 개념 정리

### 동작 방식
- [ ] `npx shadcn@latest init` / `add button` — 코드가 내 프로젝트 `components/ui/`에 복사됨
- [ ] 구성 요소: Radix(동작/접근성) + Tailwind(스타일) + CVA(variant 관리)
- [ ] 복사된 코드는 내 것 — 자유롭게 수정, 대신 업데이트는 수동

### 핵심 컴포넌트 익히기
- [ ] Button, Input, Card, Dialog, DropdownMenu, Form, Toast(sonner)
- [ ] variant 시스템 (CVA): `variant="destructive" size="sm"`
- [ ] Form + react-hook-form + zod 조합 — 검증 포함 폼의 표준 패턴

### 테마
- [ ] CSS 변수 기반 색상 시스템 (`--primary`, `--background`…)
- [ ] 다크 모드 (`next-themes` 연동)
- [ ] 서버 컴포넌트 호환 — 어떤 컴포넌트가 클라이언트 컴포넌트인가 확인하기

### 판단 기준
- [ ] shadcn이 맞는 경우 vs MUI 같은 완제품이 맞는 경우 (커스터마이징 요구 vs 개발 속도)

## 실습 (practice 앱에서)

- [ ] shadcn 셋업, 메모 UI를 Card/Button/Input으로 교체
- [ ] 메모 삭제에 Dialog(확인 모달), 완료 알림에 Toast 적용
- [ ] 메모 작성 폼을 Form + zod 검증으로 재구현 (03장 Server Action과 연결)
- [ ] `next-themes`로 다크 모드 완성 (06장 토글을 시스템 연동으로 업그레이드)

## 참고 자료

- https://ui.shadcn.com/docs
