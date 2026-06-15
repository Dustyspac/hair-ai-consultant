# 03_admin_crud.md

## 목적

매장 실제 시술 메뉴를 등록하는 관리자 CRUD 기능 구현을 위한 Codex 프롬프트입니다.

## 프롬프트

관리자용 시술 메뉴 CRUD 기능을 구현해줘.

프로젝트:

헤어샵 태블릿용 AI 상담 리포트 앱

목표:

관리자가 매장의 실제 시술 메뉴를 등록하고, 추천 엔진이 해당 메뉴를 기반으로 추천할 수 있게 한다.

기술:

- React
- TypeScript
- Tailwind CSS
- 현재는 로컬 상태 또는 sample data 기반
- 추후 로컬 DB로 확장 가능하게 설계

TreatmentMenu 타입:

- menuId: string
- storeId: string
- name: string
- category: 'cut' | 'perm' | 'color' | 'clinic' | 'scalp' | 'other'
- price: number
- durationMinutes: number
- recommendedFor: string[]
- avoidFor: string[]
- upsellMenuIds: string[]
- priority: number
- designerScript: string
- isActive: boolean
- createdAt: string
- updatedAt: string

구현할 화면:

- AdminMenuListScreen
- AdminMenuFormScreen
- AdminMenuDetailScreen

기능:

- 메뉴 목록 보기
- 메뉴 추가
- 메뉴 수정
- 메뉴 비활성화
- 메뉴 삭제
- 카테고리 필터
- 가격 입력
- 소요시간 입력
- 추천 조건 입력
- 비추천 조건 입력
- 연결 추천 메뉴 설정
- 디자이너 상담 멘트 입력

주의:

- 실제 서버 연동은 하지 않는다.
- 삭제 시 confirm 창 또는 확인 UI를 넣는다.
- 가격과 소요시간은 숫자 유효성 검사를 한다.
- 빈 메뉴명 저장 불가.
- 추천 조건은 문자열 배열로 관리한다.
- 추천 엔진이 menuId를 기준으로 참조할 수 있게 한다.

완료 기준:

- 샘플 메뉴 목록 표시
- 신규 메뉴 추가 가능
- 기존 메뉴 수정 가능
- 메뉴 삭제 또는 비활성화 가능
- TypeScript 에러 없음
