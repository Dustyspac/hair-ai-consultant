# 04_local_db.md

## 목적

태블릿 로컬 환경에서 매장 메뉴, 상담 세션, 동의 기록, 추천 결과, 삭제 로그를 저장하기 위한 Codex 프롬프트입니다.

## 프롬프트

로컬 DB 레이어를 구현해줘.

프로젝트:

헤어샵 태블릿용 AI 상담 리포트 앱

목표:

1차 MVP는 로컬 우선 구조다.
매장 메뉴, 고객 상담 세션, 동의 기록, 추천 결과, 삭제 로그를 로컬에 저장해야 한다.

기술:

- React
- TypeScript
- IndexedDB 또는 localForage 사용
- 추후 SQLite/Capacitor로 전환 가능하게 추상화

저장할 엔티티:

- Store
- TreatmentMenu
- ConsultationSession
- ConsentRecord
- FaceAnalysisResult
- RecommendationResult
- ConsultationReport
- DeletionLog

요구사항:

- src/db 폴더에 DB 접근 레이어 생성
- 엔티티별 CRUD 함수 생성
- UI 컴포넌트에서 직접 IndexedDB를 호출하지 않게 분리
- 저장 동의 없는 세션 삭제 함수 제공
- 삭제 로그 생성 함수 제공
- 타입은 src/types/index.ts를 사용
- 에러 처리 포함

필수 함수 예시:

- createTreatmentMenu
- updateTreatmentMenu
- deleteTreatmentMenu
- createConsultationSession
- saveConsentRecord
- saveFaceAnalysisResult
- saveRecommendationResult
- saveConsultationReport
- deleteSessionData
- createDeletionLog

주의:

- 고객 이름과 전화번호 필드는 만들지 않는다.
- 원본 얼굴 사진 저장은 기본 구현하지 않는다.
- 저장 동의 없는 데이터는 삭제 가능해야 한다.
- 삭제 로그에는 민감한 얼굴 분석값을 넣지 않는다.
- 외부 서버 전송 로직은 구현하지 않는다.

완료 기준:

- 매장 메뉴를 로컬 DB에 저장/조회 가능
- 상담 세션 생성 가능
- 동의 기록 저장 가능
- 세션 데이터 삭제 함수 작동
- 삭제 로그 생성 가능
- TypeScript 에러 없음
