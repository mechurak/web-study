# 06. Tailwind CSS — 유틸리티 퍼스트 스타일링

> **핵심 질문**
> - 유틸리티 퍼스트란? CSS 파일/CSS-in-JS 방식과 비교해 무엇이 좋고 무엇이 불편한가?
> - "클래스가 지저분해진다"는 비판에 대한 답은? (컴포넌트로 추상화, 디자인 시스템)
> - Tailwind는 어떻게 빌드 시 사용한 클래스만 남기는가? (JIT, content 스캔)

## 개념 정리

### 기본기
- [ ] 유틸리티 클래스 체계: spacing(`p-4`, `mx-auto`), 색상(`bg-slate-100`), 타이포(`text-sm font-medium`)
- [ ] Flexbox/Grid 레이아웃 유틸리티
- [ ] 상태 variant: `hover:`, `focus:`, `disabled:`, `group-hover:`

### 반응형 & 다크 모드
- [ ] 모바일 퍼스트 브레이크포인트 (`sm: md: lg:`)
- [ ] `dark:` variant — 다크 모드 전략 (class vs media)

### 설정과 확장
- [ ] 설정 파일 — 디자인 토큰(색상/폰트/spacing) 커스터마이징
- [ ] `@apply`는 언제 쓰고 언제 피하나
- [ ] CSS 변수와 함께 쓰기 (shadcn 테마의 기반)

### 유지보수 패턴
- [ ] 클래스 정렬 (prettier-plugin-tailwindcss)
- [ ] 조건부 클래스: `clsx` + `tailwind-merge` (`cn()` 헬퍼) — shadcn에서 표준으로 쓰는 이유

## 실습 (practice 앱에서)

- [ ] 기존 페이지들의 스타일을 Tailwind로 정리 (create-next-app에 기본 포함)
- [ ] 메모 목록을 카드 그리드로 — 반응형 (모바일 1열 → 데스크톱 3열)
- [ ] 다크 모드 토글 구현
- [ ] `cn()` 헬퍼 만들어 조건부 스타일 (선택된 메모 하이라이트)

## 참고 자료

- https://tailwindcss.com/docs
