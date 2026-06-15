# 02_ui_components.md

## 목적

태블릿용 UI 컴포넌트와 고객 플로우 화면을 구현하기 위한 Codex 프롬프트입니다.

## 프롬프트

헤어샵 태블릿용 AI 상담 리포트 앱의 공통 UI 컴포넌트를 구현해줘.

기술:

- React
- TypeScript
- Tailwind CSS

목표:

태블릿 가로 화면에서 사용하기 편한 큰 버튼, 카드형 선택 UI, 하단 진행 버튼, 상단 단계 표시를 만든다.

구현할 컴포넌트:

- AppLayout
- StepHeader
- PrimaryButton
- SecondaryButton
- SelectCard
- MultiSelectCardGroup
- NoticeBox
- ProgressFooter

적용할 화면:

- StartScreen
- TreatmentSelectScreen
- ConcernSelectScreen
- DesiredImageSelectScreen

요구사항:

- 터치하기 쉬운 큰 버튼 사용
- 고객이 선택한 항목은 명확히 강조
- 다음 버튼은 필수 선택값이 없으면 비활성화
- 태블릿 가로 화면 기준
- 접근성 기본 속성 적용
- 하드코딩 최소화
- 선택값은 상위 상태로 전달 가능하게 설계

주의:

- 아직 로컬 DB 저장은 구현하지 않는다.
- 실제 얼굴 분석은 구현하지 않는다.
- 디자인은 미니멀하게 구성한다.
- 개인정보 동의 이전에는 촬영 화면으로 이동할 수 없어야 한다.

완료 기준:

- 시술 메뉴 다중 선택 가능
- 고객 고민 다중 선택 가능
- 원하는 이미지 다중 선택 가능
- 다음 단계로 이동 가능
- TypeScript 에러 없음
