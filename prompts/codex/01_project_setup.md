# 01_project_setup.md

## 목적

프로젝트 초기 세팅을 자동화하기 위한 Codex 프롬프트입니다.

## 프롬프트

React + TypeScript + Vite 기반 태블릿 웹앱 프로젝트를 초기화해줘.

프로젝트 목적:

헤어샵 태블릿용 AI 상담 리포트 앱이다.
고객이 시술 메뉴와 고민을 선택하고, 개인정보 동의 후 얼굴 분석 결과를 바탕으로 매장 실제 메뉴를 추천받는 구조다.

요구사항:

- React
- TypeScript
- Vite
- Tailwind CSS
- 기본 라우팅 구조
- 태블릿 가로 화면 기준 UI
- src/features 구조 생성
- docs 폴더는 유지
- prompts 폴더는 유지
- npm run dev로 실행 가능해야 함

생성할 구조:

src/
app/
components/
features/
customer-flow/
admin/
consent/
camera/
face-analysis/
recommendation/
report/
data/
db/
lib/
types/

필수 생성 파일:

- src/types/index.ts
- src/data/sampleMenus.ts
- src/app/App.tsx
- src/app/routes.tsx 또는 라우팅 관련 파일
- src/features/customer-flow/StartScreen.tsx
- src/features/customer-flow/TreatmentSelectScreen.tsx
- src/features/customer-flow/ConcernSelectScreen.tsx
- src/features/admin/AdminHomeScreen.tsx

주의:

- 실제 얼굴 분석 기능은 아직 구현하지 않는다.
- 실제 카메라 기능은 아직 구현하지 않는다.
- 실제 서버 연동은 하지 않는다.
- UI는 단순해도 된다.
- TypeScript 에러가 없어야 한다.
- 실행 방법을 README에 적어줘.

완료 기준:

- npm install 후 npm run dev 실행 가능
- 시작 화면 진입 가능
- 시술 메뉴 선택 화면 이동 가능
- 고객 고민 선택 화면 이동 가능
- 관리자 홈 화면 진입 가능
- TypeScript 에러 없음
