# 헤어샵 AI 상담 리포트 MVP 데이터 구조 설계

## 0. 설계 전제

- 1차 MVP는 **로컬 DB 기준**으로 설계한다.
- 고객 이름과 전화번호는 **기본적으로 수집하지 않는다**.
- 원본 사진은 DB에 직접 저장하지 않고, **로컬 파일 경로만 참조**한다.
- 저장 동의가 없는 경우, 상담 종료 후 **원본 사진과 얼굴 분석 결과를 삭제**한다.
- 추천 결과는 반드시 매장에 등록된 **실제 시술 메뉴 ID**와 연결한다.
- 모든 데이터 구조는 TypeScript 타입으로 바로 변환 가능한 형태를 기준으로 한다.
- AI 분석 결과와 추천 리포트는 **실제 시술 결과를 보장하지 않는 상담 참고용 데이터**로 취급한다.

---

# 1. 핵심 엔티티 목록

| 엔티티                 | 목적                                   | 주요 연결                                            |
| ---------------------- | -------------------------------------- | ---------------------------------------------------- |
| `StoreInfo`            | 매장 기본 정보와 로컬 설정 저장        | `TreatmentMenu`, `ConsultationSession`               |
| `TreatmentMenu`        | 매장 실제 시술 메뉴 저장               | `RecommendationResult.items.menuId`                  |
| `ConsultationSession`  | 고객 1회 상담 단위                     | 동의, 사진, 분석, 추천, 리포트의 기준                |
| `ConsentRecord`        | 개인정보·사진 촬영·분석·저장 동의 기록 | `sessionId`                                          |
| `PhotoAsset`           | 원본 사진의 로컬 파일 참조             | `sessionId`, `FaceAnalysisResult.sourcePhotoAssetId` |
| `FaceAnalysisResult`   | 얼굴 분석 결과                         | `sessionId`, `photoAssetId`                          |
| `RecommendationResult` | 매장 메뉴 기반 추천 결과               | `menuId`, `analysisId`                               |
| `ConsultationReport`   | 상담용 리포트 데이터                   | `sessionId`, `recommendationResultId`                |
| `DeletionLog`          | 삭제 수행 기록                         | `sessionId`, 삭제 대상 목록                          |

> `PhotoAsset`은 원본 사진의 저장·삭제 상태를 명확하게 관리하기 위해 별도 엔티티로 둔다.

---

# 2. 각 엔티티의 JSON 예시

## 2-1. 매장 정보: `StoreInfo`

```json
{
	"id": "store_001",
	"name": "라움헤어 둔산점",
	"timezone": "Asia/Seoul",
	"currency": "KRW",
	"address": "대전 서구 둔산동",
	"contactNumber": null,
	"businessRegistrationNumber": null,
	"settings": {
		"defaultPhotoRetention": "delete_after_session",
		"defaultAnalysisRetention": "delete_after_session",
		"allowReportSave": true,
		"requireConsentBeforePhoto": true,
		"showDisclaimerOnEveryReport": true
	},
	"createdAt": "2026-06-14T10:00:00.000+09:00",
	"updatedAt": "2026-06-14T10:00:00.000+09:00"
}
```

---

## 2-2. 시술 메뉴: `TreatmentMenu`

```json
{
	"id": "menu_cut_layer_001",
	"storeId": "store_001",
	"category": "cut",
	"name": "레이어드 커트",
	"description": "얼굴형 보완과 볼륨감을 위한 레이어드 커트",
	"price": 45000,
	"durationMinutes": 60,
	"isActive": true,
	"tags": ["볼륨", "얼굴형보완", "여성커트"],
	"recommendationRules": {
		"faceShapes": ["round", "square"],
		"concerns": ["wide_face", "flat_volume"],
		"desiredImages": ["soft", "natural"],
		"hairLengths": ["medium", "long"],
		"excludedConditions": []
	},
	"displayOrder": 1,
	"createdAt": "2026-06-14T10:00:00.000+09:00",
	"updatedAt": "2026-06-14T10:00:00.000+09:00"
}
```

---

## 2-3. 고객 상담 세션: `ConsultationSession`

```json
{
	"id": "session_20260614_0001",
	"storeId": "store_001",
	"status": "report_ready",
	"startedAt": "2026-06-14T14:05:00.000+09:00",
	"endedAt": null,
	"deviceId": "tablet_local_001",
	"customerAlias": "현장고객-0001",
	"customerInput": {
		"selectedServiceCategories": ["cut", "perm"],
		"concerns": ["wide_face", "flat_volume"],
		"desiredImages": ["soft", "natural"],
		"currentHairLength": "medium",
		"chemicalHistory": ["perm_6_months_ago"],
		"memoByDesigner": "볼륨이 쉽게 죽는 편이라고 상담 중 언급"
	},
	"consentId": "consent_20260614_0001",
	"photoAssetIds": ["photo_20260614_0001_front"],
	"analysisResultId": "analysis_20260614_0001",
	"recommendationResultId": "recommend_20260614_0001",
	"reportId": "report_20260614_0001",
	"retentionPolicy": {
		"saveOriginalPhoto": false,
		"saveAnalysisResult": false,
		"saveReport": true,
		"deleteAtSessionEnd": true
	},
	"createdAt": "2026-06-14T14:05:00.000+09:00",
	"updatedAt": "2026-06-14T14:12:00.000+09:00"
}
```

---

## 2-4. 개인정보 동의: `ConsentRecord`

```json
{
	"id": "consent_20260614_0001",
	"sessionId": "session_20260614_0001",
	"storeId": "store_001",
	"consentVersion": "2026-06-01",
	"agreedAt": "2026-06-14T14:06:00.000+09:00",
	"agreedBy": "customer",
	"confirmedBy": "designer",
	"items": {
		"photoCapture": true,
		"faceAnalysis": true,
		"useForConsultation": true,
		"saveOriginalPhoto": false,
		"saveAnalysisResult": false,
		"saveReport": true,
		"marketingUse": false
	},
	"noticeTextSnapshot": {
		"privacyNotice": "촬영된 사진은 상담 목적으로만 사용되며 저장 동의가 없는 경우 상담 종료 후 삭제됩니다.",
		"aiDisclaimer": "AI 분석과 추천 이미지는 실제 시술 결과를 보장하지 않으며 상담 참고용입니다."
	},
	"createdAt": "2026-06-14T14:06:00.000+09:00"
}
```

---

## 2-5. 사진 자산: `PhotoAsset`

```json
{
	"id": "photo_20260614_0001_front",
	"sessionId": "session_20260614_0001",
	"storeId": "store_001",
	"type": "front_face",
	"localUri": "app-local://photos/session_20260614_0001/front.jpg",
	"mimeType": "image/jpeg",
	"fileSizeBytes": 824120,
	"width": 1280,
	"height": 960,
	"capturedAt": "2026-06-14T14:07:00.000+09:00",
	"storageStatus": "temporary",
	"deleteAfterSession": true,
	"deletedAt": null
}
```

---

## 2-6. 얼굴 분석 결과: `FaceAnalysisResult`

```json
{
	"id": "analysis_20260614_0001",
	"sessionId": "session_20260614_0001",
	"storeId": "store_001",
	"sourcePhotoAssetId": "photo_20260614_0001_front",
	"status": "completed",
	"analyzedAt": "2026-06-14T14:08:00.000+09:00",
	"modelInfo": {
		"provider": "local",
		"modelName": "face-consult-mvp",
		"modelVersion": "0.1.0"
	},
	"result": {
		"faceShape": "round",
		"faceShapeConfidence": 0.78,
		"proportions": {
			"midfaceRatio": "slightly_long",
			"foreheadWidth": "normal",
			"cheekboneWidth": "wide",
			"jawline": "soft"
		},
		"styleConsiderations": [
			"옆볼륨을 과하게 키우는 스타일은 얼굴 폭이 더 넓어 보일 수 있음",
			"정수리 볼륨과 세로 흐름을 만드는 스타일이 적합할 수 있음"
		],
		"riskNotes": ["조명, 촬영 각도, 표정에 따라 분석 결과가 달라질 수 있음"]
	},
	"rawLandmarksStored": false,
	"faceEmbeddingStored": false,
	"deleteAfterSession": true,
	"deletedAt": null
}
```

---

## 2-7. 추천 결과: `RecommendationResult`

```json
{
	"id": "recommend_20260614_0001",
	"sessionId": "session_20260614_0001",
	"storeId": "store_001",
	"analysisResultId": "analysis_20260614_0001",
	"createdAt": "2026-06-14T14:09:00.000+09:00",
	"items": [
		{
			"rank": 1,
			"menuId": "menu_cut_layer_001",
			"menuSnapshot": {
				"name": "레이어드 커트",
				"category": "cut",
				"price": 45000,
				"durationMinutes": 60
			},
			"reasonCodes": ["face_shape_balance", "volume_control", "soft_image"],
			"reasonText": "둥근 얼굴형과 볼륨 고민을 고려했을 때 세로 흐름과 정수리 볼륨을 만들기 쉬운 메뉴입니다.",
			"consultationPoints": [
				"앞머리 유무",
				"레이어 높이",
				"얼굴 옆선 커버 정도"
			],
			"disclaimer": "추천은 상담 참고용이며 실제 결과를 보장하지 않습니다."
		},
		{
			"rank": 2,
			"menuId": "menu_perm_root_001",
			"menuSnapshot": {
				"name": "뿌리 볼륨펌",
				"category": "perm",
				"price": 70000,
				"durationMinutes": 90
			},
			"reasonCodes": ["flat_volume", "top_volume"],
			"reasonText": "정수리 볼륨을 보완해 얼굴이 세로로 길어 보이는 인상을 줄 수 있습니다.",
			"consultationPoints": ["모발 손상도", "최근 펌 이력", "볼륨 유지 기간"],
			"disclaimer": "추천은 상담 참고용이며 실제 결과를 보장하지 않습니다."
		}
	],
	"summaryText": "레이어드 커트와 뿌리 볼륨펌 조합을 우선 상담하는 것이 적합합니다.",
	"deleteAfterSession": false,
	"deletedAt": null
}
```

---

## 2-8. 상담 리포트: `ConsultationReport`

```json
{
	"id": "report_20260614_0001",
	"sessionId": "session_20260614_0001",
	"storeId": "store_001",
	"recommendationResultId": "recommend_20260614_0001",
	"title": "AI 헤어 상담 리포트",
	"status": "ready",
	"createdAt": "2026-06-14T14:10:00.000+09:00",
	"sections": {
		"customerSummary": "고객은 부드럽고 자연스러운 이미지를 원하며, 얼굴 폭과 볼륨 부족을 고민으로 선택했습니다.",
		"analysisSummary": "둥근 얼굴형 경향과 넓은 광대 라인이 감지되었습니다.",
		"recommendationSummary": "레이어드 커트와 뿌리 볼륨펌을 우선 추천합니다.",
		"designerMemo": "실제 모발 상태 확인 후 펌 가능 여부 판단 필요"
	},
	"linkedMenuIds": ["menu_cut_layer_001", "menu_perm_root_001"],
	"disclaimer": "본 리포트는 AI 기반 상담 참고 자료이며 실제 시술 결과를 보장하지 않습니다.",
	"savedByConsent": true,
	"exportedFileUri": null,
	"deletedAt": null
}
```

---

## 2-9. 삭제 로그: `DeletionLog`

```json
{
	"id": "delete_log_20260614_0001",
	"storeId": "store_001",
	"sessionId": "session_20260614_0001",
	"reason": "no_storage_consent",
	"requestedBy": "system",
	"deletedAt": "2026-06-14T14:30:00.000+09:00",
	"deletedTargets": [
		{
			"entityType": "PhotoAsset",
			"entityId": "photo_20260614_0001_front",
			"deleteStatus": "deleted"
		},
		{
			"entityType": "FaceAnalysisResult",
			"entityId": "analysis_20260614_0001",
			"deleteStatus": "deleted"
		}
	],
	"retainedData": [
		"DeletionLog",
		"ConsentRecord",
		"ConsultationSession_minimal"
	],
	"note": "저장 동의가 없어 원본 사진과 얼굴 분석 결과를 상담 종료 후 삭제함"
}
```

---

# 3. TypeScript Interface

```ts
export type UUID = string;
export type ISODateString = string;
export type LocalUri = string;

export type CurrencyCode = "KRW";

export type SessionStatus =
	| "draft"
	| "consent_pending"
	| "photo_ready"
	| "analyzing"
	| "analysis_ready"
	| "report_ready"
	| "completed"
	| "deleted";

export type MenuCategory =
	| "cut"
	| "perm"
	| "color"
	| "clinic"
	| "styling"
	| "other";

export type FaceShape =
	| "round"
	| "oval"
	| "square"
	| "long"
	| "heart"
	| "diamond"
	| "unknown";

export type CustomerConcern =
	| "wide_face"
	| "long_face"
	| "flat_volume"
	| "damaged_hair"
	| "frizzy_hair"
	| "thin_hair"
	| "strong_jawline"
	| "wide_cheekbone"
	| "other";

export type DesiredImage =
	| "soft"
	| "natural"
	| "clean"
	| "trendy"
	| "mature"
	| "cute"
	| "professional"
	| "other";

export type HairLength = "short" | "medium" | "long" | "unknown";

export interface StoreInfo {
	id: UUID;
	name: string;
	timezone: "Asia/Seoul";
	currency: CurrencyCode;
	address?: string | null;
	contactNumber?: string | null;
	businessRegistrationNumber?: string | null;
	settings: StoreSettings;
	createdAt: ISODateString;
	updatedAt: ISODateString;
}

export interface StoreSettings {
	defaultPhotoRetention: "delete_after_session" | "saved_by_consent";
	defaultAnalysisRetention: "delete_after_session" | "saved_by_consent";
	allowReportSave: boolean;
	requireConsentBeforePhoto: boolean;
	showDisclaimerOnEveryReport: boolean;
}

export interface TreatmentMenu {
	id: UUID;
	storeId: UUID;
	category: MenuCategory;
	name: string;
	description?: string;
	price: number;
	durationMinutes: number;
	isActive: boolean;
	tags: string[];
	recommendationRules: RecommendationRule;
	displayOrder: number;
	createdAt: ISODateString;
	updatedAt: ISODateString;
}

export interface RecommendationRule {
	faceShapes: FaceShape[];
	concerns: CustomerConcern[];
	desiredImages: DesiredImage[];
	hairLengths: HairLength[];
	excludedConditions: string[];
}

export interface ConsultationSession {
	id: UUID;
	storeId: UUID;
	status: SessionStatus;
	startedAt: ISODateString;
	endedAt?: ISODateString | null;
	deviceId: string;

	/**
	 * 고객 실명 금지.
	 * 현장 구분용 임시 별칭만 사용.
	 */
	customerAlias?: string | null;

	customerInput: SessionCustomerInput;

	consentId?: UUID | null;
	photoAssetIds: UUID[];
	analysisResultId?: UUID | null;
	recommendationResultId?: UUID | null;
	reportId?: UUID | null;

	retentionPolicy: SessionRetentionPolicy;

	createdAt: ISODateString;
	updatedAt: ISODateString;
}

export interface SessionCustomerInput {
	selectedServiceCategories: MenuCategory[];
	concerns: CustomerConcern[];
	desiredImages: DesiredImage[];
	currentHairLength: HairLength;
	chemicalHistory: string[];
	memoByDesigner?: string | null;
}

export interface SessionRetentionPolicy {
	saveOriginalPhoto: boolean;
	saveAnalysisResult: boolean;
	saveReport: boolean;
	deleteAtSessionEnd: boolean;
}

export interface ConsentRecord {
	id: UUID;
	sessionId: UUID;
	storeId: UUID;
	consentVersion: string;
	agreedAt: ISODateString;
	agreedBy: "customer";
	confirmedBy: "designer" | "system";
	items: ConsentItems;
	noticeTextSnapshot: ConsentNoticeSnapshot;
	createdAt: ISODateString;
}

export interface ConsentItems {
	photoCapture: boolean;
	faceAnalysis: boolean;
	useForConsultation: boolean;
	saveOriginalPhoto: boolean;
	saveAnalysisResult: boolean;
	saveReport: boolean;
	marketingUse: false;
}

export interface ConsentNoticeSnapshot {
	privacyNotice: string;
	aiDisclaimer: string;
}

export interface PhotoAsset {
	id: UUID;
	sessionId: UUID;
	storeId: UUID;
	type: "front_face" | "side_face" | "hair_reference" | "other";
	localUri: LocalUri;
	mimeType: "image/jpeg" | "image/png" | "image/webp";
	fileSizeBytes: number;
	width: number;
	height: number;
	capturedAt: ISODateString;
	storageStatus: "temporary" | "saved" | "deleted";
	deleteAfterSession: boolean;
	deletedAt?: ISODateString | null;
}

export interface FaceAnalysisResult {
	id: UUID;
	sessionId: UUID;
	storeId: UUID;
	sourcePhotoAssetId: UUID;
	status: "pending" | "completed" | "failed" | "deleted";
	analyzedAt?: ISODateString | null;
	modelInfo: AnalysisModelInfo;
	result?: FaceAnalysisPayload | null;

	/**
	 * MVP에서는 반드시 false 권장.
	 */
	rawLandmarksStored: false;
	faceEmbeddingStored: false;

	deleteAfterSession: boolean;
	deletedAt?: ISODateString | null;
}

export interface AnalysisModelInfo {
	provider: "local" | "external_api";
	modelName: string;
	modelVersion: string;
}

export interface FaceAnalysisPayload {
	faceShape: FaceShape;
	faceShapeConfidence: number;
	proportions: FaceProportions;
	styleConsiderations: string[];
	riskNotes: string[];
}

export interface FaceProportions {
	midfaceRatio: "short" | "normal" | "slightly_long" | "long" | "unknown";
	foreheadWidth: "narrow" | "normal" | "wide" | "unknown";
	cheekboneWidth: "narrow" | "normal" | "wide" | "unknown";
	jawline: "soft" | "angular" | "strong" | "unknown";
}

export interface RecommendationResult {
	id: UUID;
	sessionId: UUID;
	storeId: UUID;
	analysisResultId?: UUID | null;
	createdAt: ISODateString;
	items: RecommendedMenuItem[];
	summaryText: string;
	deleteAfterSession: boolean;
	deletedAt?: ISODateString | null;
}

export interface RecommendedMenuItem {
	rank: number;
	menuId: UUID;

	/**
	 * 메뉴 가격·이름 변경에 대비한 상담 당시 스냅샷.
	 * 실제 연결 기준은 menuId.
	 */
	menuSnapshot: MenuSnapshot;

	reasonCodes: string[];
	reasonText: string;
	consultationPoints: string[];
	disclaimer: string;
}

export interface MenuSnapshot {
	name: string;
	category: MenuCategory;
	price: number;
	durationMinutes: number;
}

export interface ConsultationReport {
	id: UUID;
	sessionId: UUID;
	storeId: UUID;
	recommendationResultId: UUID;
	title: string;
	status: "draft" | "ready" | "exported" | "deleted";
	createdAt: ISODateString;
	sections: ReportSections;
	linkedMenuIds: UUID[];
	disclaimer: string;
	savedByConsent: boolean;
	exportedFileUri?: LocalUri | null;
	deletedAt?: ISODateString | null;
}

export interface ReportSections {
	customerSummary: string;
	analysisSummary: string;
	recommendationSummary: string;
	designerMemo?: string | null;
}

export interface DeletionLog {
	id: UUID;
	storeId: UUID;
	sessionId?: UUID | null;
	reason:
		| "no_storage_consent"
		| "customer_delete_request"
		| "retention_expired"
		| "admin_delete"
		| "system_cleanup";
	requestedBy: "customer" | "designer" | "admin" | "system";
	deletedAt: ISODateString;
	deletedTargets: DeletedTarget[];
	retainedData: string[];
	note?: string | null;
}

export interface DeletedTarget {
	entityType:
		| "PhotoAsset"
		| "FaceAnalysisResult"
		| "RecommendationResult"
		| "ConsultationReport"
		| "ConsultationSession";
	entityId: UUID;
	deleteStatus: "deleted" | "not_found" | "failed";
}
```

---

# 4. 저장하면 안 되는 데이터

| 구분                         | 저장 금지 데이터                                                   | 이유                               |
| ---------------------------- | ------------------------------------------------------------------ | ---------------------------------- |
| 고객 식별정보                | 고객 이름, 전화번호, 생년월일, 주소, SNS ID                        | MVP 기본 조건상 수집하지 않음      |
| 생체 식별 데이터             | 얼굴 임베딩, 얼굴 인식용 벡터, 고유 식별 템플릿                    | 상담 목적을 넘어서는 고위험 데이터 |
| 원본 분석 원천값             | 전체 랜드마크 좌표, 원본 얼굴 매핑 데이터, 디버깅용 원본 추론 로그 | 재식별 위험 증가                   |
| 원본 사진 복제본             | Base64 이미지 문자열, 썸네일 무단 저장, 리포트 내부 이미지 복사본  | 삭제 누락 위험                     |
| 불필요한 민감 정보           | 두피 질환, 질병명, 약물 복용, 임신 여부 등                         | 헤어 상담 MVP 범위 초과            |
| 외부 연동 보안값             | API Key, Access Token, Refresh Token 평문 저장                     | 보안 사고 위험                     |
| 마케팅 동의 없는 활용 데이터 | 광고 타겟팅용 고객 프로필, 재방문 추적 ID                          | MVP 목적과 불일치                  |
| 과장 표현 데이터             | “어울림 보장”, “시술 결과 예측 확정”, “얼굴형 확정 진단”           | 클레임 리스크 증가                 |

---

# 5. 삭제 시 함께 삭제해야 하는 데이터

## 5-1. 저장 동의가 없는 경우

상담 종료 시 자동 삭제해야 한다.

| 기준 데이터                     | 함께 삭제                                   |
| ------------------------------- | ------------------------------------------- |
| `PhotoAsset`                    | 로컬 원본 사진 파일, 썸네일, 임시 캐시      |
| `FaceAnalysisResult`            | 얼굴형 분석 결과, 비율 분석 결과, 분석 요약 |
| 임시 AI 생성 이미지가 있는 경우 | 생성 이미지 파일, 프리뷰 이미지, 캐시       |
| 임시 리포트 파일                | PDF, 이미지 export 파일, 공유용 임시 파일   |

남겨도 되는 데이터는 최소화해야 한다.

| 보존 가능 데이터                | 조건                                                            |
| ------------------------------- | --------------------------------------------------------------- |
| `DeletionLog`                   | 삭제 사실만 기록, 사진 경로·분석 내용 저장 금지                 |
| `ConsentRecord`                 | 동의 여부 증빙 목적                                             |
| `ConsultationSession` 최소 정보 | `status`, `startedAt`, `endedAt`, `retentionPolicy` 정도만 보존 |

---

## 5-2. 고객이 전체 삭제를 요청한 경우

아래 데이터를 함께 삭제해야 한다.

| 삭제 대상              |
| ---------------------- |
| `ConsultationSession`  |
| `ConsentRecord`        |
| `PhotoAsset`           |
| `FaceAnalysisResult`   |
| `RecommendationResult` |
| `ConsultationReport`   |
| export 파일            |
| 임시 캐시              |
| 썸네일                 |
| AI 생성 이미지         |
| 로컬 검색 인덱스       |

삭제 이력을 남겨야 한다면 `DeletionLog`에는 삭제 사실만 남긴다.

```json
{
	"reason": "customer_delete_request",
	"deletedAt": "2026-06-14T15:00:00.000+09:00",
	"deletedTargets": [
		{
			"entityType": "ConsultationSession",
			"entityId": "session_20260614_0001",
			"deleteStatus": "deleted"
		}
	],
	"note": "고객 요청으로 상담 데이터 전체 삭제"
}
```

---

## 5-3. 추천 결과 삭제 기준

| 상황                     | 처리                                                                |
| ------------------------ | ------------------------------------------------------------------- |
| 분석 결과 저장 동의 없음 | 추천 결과만 남길지 여부를 별도 동의로 분리하는 것이 안전            |
| 리포트 저장 동의 있음    | 추천 메뉴 ID와 상담 요약은 보존 가능                                |
| 전체 삭제 요청           | 추천 결과도 삭제                                                    |
| 메뉴가 나중에 변경됨     | `menuId`는 유지하되 `menuSnapshot`으로 상담 당시 가격·소요시간 보존 |

---

# 6. 나중에 서버 연동 시 확장해야 할 필드

## 6-1. 모든 엔티티 공통 확장 필드

```ts
export interface ServerSyncFields {
	remoteId?: string | null;
	tenantId?: string;
	syncStatus?: "local_only" | "pending" | "synced" | "failed" | "conflict";
	lastSyncedAt?: ISODateString | null;
	createdByDeviceId?: string;
	updatedByDeviceId?: string;
	schemaVersion?: number;
	version?: number;
	deletedAt?: ISODateString | null;
}
```

---

## 6-2. 서버 연동 시 추가할 필드 목록

| 영역        | 확장 필드                                             | 목적                      |
| ----------- | ----------------------------------------------------- | ------------------------- |
| 매장 구분   | `tenantId`, `storeGroupId`                            | 여러 매장·프랜차이즈 관리 |
| 기기 구분   | `deviceId`, `deviceName`, `registeredAt`              | 태블릿별 동기화 관리      |
| 사용자 권한 | `createdByUserId`, `role`, `designerId`, `adminId`    | 디자이너·관리자 권한 분리 |
| 동기화      | `remoteId`, `syncStatus`, `lastSyncedAt`              | 로컬-서버 데이터 동기화   |
| 충돌 해결   | `version`, `updatedAt`, `conflictResolutionPolicy`    | 오프라인 수정 충돌 처리   |
| 보안        | `encryptedAt`, `encryptionKeyId`                      | 서버 저장 시 암호화 추적  |
| 감사 로그   | `auditLogId`, `createdBy`, `deletedBy`                | 삭제·수정 이력 관리       |
| 동의 관리   | `consentVersion`, `policyVersion`, `noticeHash`       | 약관 변경 대응            |
| 데이터 보존 | `retentionExpiresAt`, `autoDeleteScheduledAt`         | 자동 삭제 예약            |
| 파일 업로드 | `remoteFileUri`, `uploadStatus`, `checksum`           | 사진·PDF 파일 서버 업로드 |
| 리포트 공유 | `shareToken`, `shareExpiresAt`, `viewCount`           | 고객 공유 링크 기능       |
| 과금        | `subscriptionPlanId`, `usageCount`, `billingMonth`    | SaaS 과금 모델            |
| 분석 품질   | `modelVersion`, `analysisProvider`, `confidenceScore` | 모델 변경 추적            |

---

# 7. 권장 로컬 DB 테이블 구조

```txt
stores
treatment_menus
consultation_sessions
consent_records
photo_assets
face_analysis_results
recommendation_results
consultation_reports
deletion_logs
```

---

# 8. 엔티티 관계

```txt
StoreInfo 1 ── N TreatmentMenu
StoreInfo 1 ── N ConsultationSession

ConsultationSession 1 ── 1 ConsentRecord
ConsultationSession 1 ── N PhotoAsset
ConsultationSession 1 ── 1 FaceAnalysisResult
ConsultationSession 1 ── 1 RecommendationResult
ConsultationSession 1 ── 1 ConsultationReport

RecommendationResult N ── N TreatmentMenu
DeletionLog N ── 1 ConsultationSession
```

---

# 9. MVP 기준 핵심 설계 원칙

1. 고객 실명과 전화번호는 수집하지 않는다.
2. 원본 사진은 DB에 직접 저장하지 않고 로컬 파일 경로만 저장한다.
3. 저장 동의가 없으면 `PhotoAsset`, `FaceAnalysisResult`, 임시 AI 이미지, 캐시를 상담 종료 시 삭제한다.
4. 추천 결과는 반드시 `TreatmentMenu.id`와 연결한다.
5. 상담 리포트에는 “실제 결과 보장 아님” 문구를 항상 포함한다.
6. 삭제 로그에는 삭제 사실만 남기고 얼굴 분석 내용이나 사진 경로는 남기지 않는다.
7. 서버 연동을 고려해 모든 엔티티에 `storeId`, `createdAt`, `updatedAt`, `deletedAt` 구조를 유지한다.
8. AI 분석 결과는 진단이나 확정 판단이 아니라 상담 보조 정보로만 사용한다.
9. 매장 메뉴가 변경되어도 과거 상담 리포트가 깨지지 않도록 `menuSnapshot`을 함께 저장한다.
10. 로컬 DB 기준이라도 삭제 정책과 동의 이력은 MVP 단계부터 반드시 포함한다.

---

# 10. MVP 구현 우선순위

| 우선순위 | 엔티티                 | 이유                                       |
| -------- | ---------------------- | ------------------------------------------ |
| 1        | `StoreInfo`            | 매장 단위 설정 기준                        |
| 1        | `TreatmentMenu`        | 실제 메뉴 기반 추천의 핵심                 |
| 1        | `ConsultationSession`  | 상담 흐름의 중심 데이터                    |
| 1        | `ConsentRecord`        | 개인정보·사진 촬영·분석 동의 필수          |
| 1        | `PhotoAsset`           | 사진 저장·삭제 관리 필수                   |
| 1        | `FaceAnalysisResult`   | 얼굴 분석 결과 저장 구조                   |
| 1        | `RecommendationResult` | 메뉴 기반 추천 결과                        |
| 1        | `ConsultationReport`   | 디자이너 상담용 최종 산출물                |
| 1        | `DeletionLog`          | 삭제 증빙 및 클레임 방어                   |
| 2        | `ServerSyncFields`     | 서버 연동 시 확장                          |
| 2        | `AuditLog`             | 다중 사용자·관리자 기능 확장 시 필요       |
| 2        | `DesignerAccount`      | MVP 이후 디자이너별 상담 이력 관리 시 필요 |
