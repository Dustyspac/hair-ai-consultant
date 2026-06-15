# 05_recommendation_engine.md

## 목적

고객 선택값, 얼굴 분석 결과, 매장 메뉴 데이터를 바탕으로 추천 메뉴를 생성하는 규칙 기반 엔진을 구현하기 위한 Codex 프롬프트입니다.

## 프롬프트

규칙 기반 추천 엔진 v0를 구현해줘.

프로젝트:

헤어샵 태블릿용 AI 상담 리포트 앱

목표:

고객의 선택값과 얼굴 분석 결과를 바탕으로, 매장에 등록된 실제 시술 메뉴 중 추천 메뉴 2~3개를 출력한다.

입력 데이터:

- ConsultationSession
  - selectedCategories
  - concerns
  - desiredImages
  - budgetLevel
  - careLevel

- FaceAnalysisResult
  - faceShape
  - midfaceRatio
  - jawline
  - cheekbone

- TreatmentMenu[]
  - category
  - recommendedFor
  - avoidFor
  - upsellMenuIds
  - priority
  - designerScript

출력 데이터:

- RecommendationResult
  - recommendedMenus
  - avoidNotes
  - disclaimer

추천 로직:

1. 고객이 선택한 시술 카테고리에 해당하는 메뉴를 우선 필터링한다.
2. 고객 고민과 원하는 이미지가 recommendedFor에 포함되면 점수를 높인다.
3. 얼굴 분석 결과와 recommendedFor가 일치하면 점수를 높인다.
4. avoidFor에 고객 조건이 포함되면 점수를 낮추거나 제외한다.
5. priority가 높은 메뉴에 가산점을 준다.
6. 점수 상위 2~3개 메뉴를 추천한다.
7. upsellMenuIds가 있으면 연결 추천으로 표시한다.
8. 항상 상담 참고용 disclaimer를 포함한다.

주의:

- AI 이미지 생성은 구현하지 않는다.
- 결과를 보장하는 표현을 사용하지 않는다.
- 추천 사유는 안전한 문구로 생성한다.
- 최종 시술 결정은 디자이너 상담 후 진행된다는 문구를 포함한다.
- 추천 결과는 반드시 TreatmentMenu의 menuId와 연결되어야 한다.

완료 기준:

- 샘플 세션과 샘플 메뉴를 넣으면 추천 결과가 나온다.
- 추천 메뉴는 실제 TreatmentMenu의 menuId와 연결된다.
- 추천 이유가 생성된다.
- 비추천/주의 문구가 생성된다.
- 단위 테스트 또는 간단한 테스트 함수가 있다.
- TypeScript 에러 없음
