# 06_camera_face_analysis.md

## 목적

태블릿 카메라 촬영과 얼굴 분석 기초 기능 구현을 위한 Codex 프롬프트입니다.

## 프롬프트

촬영 화면과 얼굴 분석 기초 기능을 구현해줘.

프로젝트:

헤어샵 태블릿용 AI 상담 리포트 앱

목표:

디자이너가 태블릿 카메라로 고객 정면 사진을 촬영하고, 얼굴 랜드마크 기반 분석 결과를 생성한다.

기술:

- React
- TypeScript
- 브라우저 카메라 API
- MediaPipe Face Landmarker 예정
- Tailwind CSS

구현할 화면:

- CameraGuideScreen
- CameraCaptureScreen
- AnalyzingScreen
- FaceAnalysisResultScreen

기능:

- 카메라 권한 요청
- 촬영 가이드 프레임 표시
- 정면 촬영 버튼
- 촬영 이미지 미리보기
- 재촬영 버튼
- 분석 시작 버튼
- 분석 중 폴리곤 느낌의 UI 표시
- 얼굴 분석 결과 mock 출력

1차 구현 범위:

- 실제 MediaPipe 연동 전에는 mock 분석 결과를 반환한다.
- faceShape, midfaceRatio, jawline, cheekbone 값을 임시 생성한다.
- 실제 원본 사진 저장은 하지 않는다.
- 촬영 이미지는 세션 중 임시 상태로만 관리한다.

주의:

- 고객 동의가 없으면 촬영 화면에 접근할 수 없어야 한다.
- 원본 사진을 localStorage에 저장하지 않는다.
- 분석 결과에는 상담 참고용 고지 문구를 포함한다.
- 피부톤은 조명 영향이 크므로 MVP에서는 보수적으로 표현한다.
- 실제 고객 사진을 외부 API로 전송하지 않는다.

완료 기준:

- 카메라 권한 요청 가능
- 촬영 가능
- 재촬영 가능
- 분석 중 화면 표시
- mock 얼굴 분석 결과 출력
- TypeScript 에러 없음
