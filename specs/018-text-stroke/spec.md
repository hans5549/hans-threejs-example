# Feature Specification: WebGL 文字描邊效果

**Feature Branch**: `018-text-stroke`  
**Created**: 2025-12-04  
**Status**: Draft  
**Input**: User description: "實作 Three.js WebGL 文字描邊範例，展示使用 JSON 字體載入、ShapeGeometry 建立填充文字、SVGLoader 繪製描邊線條，支援多種文字方向（左到右、右到左、上到下）"

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - 觀看 3D 文字描邊效果 (Priority: P1)

使用者打開頁面後，能夠在 3D 場景中看到具有描邊效果的文字。文字同時具有半透明的填充面和清晰的描邊線條，形成類似書法或設計字體的視覺效果。

**Why this priority**: 這是功能的核心價值，展示文字描邊技術的視覺效果，沒有它就沒有意義。

**Independent Test**: 打開頁面即可看到完整渲染的 3D 描邊文字，無需任何操作即可驗證功能。

**Acceptance Scenarios**:

1. **Given** 使用者開啟頁面, **When** 字體資源載入完成, **Then** 場景中顯示具有填充和描邊效果的 3D 文字
2. **Given** 頁面已載入, **When** 使用者觀看文字, **Then** 文字填充部分呈現半透明效果，描邊部分呈現實心線條

---

### User Story 2 - 瀏覽多語言多方向文字 (Priority: P1)

使用者能夠在場景中同時看到多種語言的文字，包括英文（左到右）、希伯來文（右到左）和中文（上到下），展示系統對不同書寫方向的支援。

**Why this priority**: 多方向文字支援是本範例的核心特色，展示 Three.js 字體系統的多語言能力。

**Independent Test**: 觀察場景中三組不同方向排列的文字是否正確顯示。

**Acceptance Scenarios**:

1. **Given** 頁面載入完成, **When** 使用者觀看場景, **Then** 能看到三組不同語言/方向的描邊文字
2. **Given** 英文文字顯示, **When** 檢視排列方向, **Then** 文字從左到右水平排列
3. **Given** 希伯來文文字顯示, **When** 檢視排列方向, **Then** 文字從右到左水平排列
4. **Given** 中文文字顯示, **When** 檢視排列方向, **Then** 文字從上到下垂直排列

---

### User Story 3 - 使用軌道控制器檢視文字 (Priority: P2)

使用者能夠透過滑鼠拖曳來旋轉、縮放和平移視角，從不同角度觀察 3D 文字的描邊效果和層次感。

**Why this priority**: 互動控制是 3D 展示的標準功能，增強使用者體驗和理解度。

**Independent Test**: 使用滑鼠拖曳、滾輪、右鍵拖曳測試三種控制方式。

**Acceptance Scenarios**:

1. **Given** 頁面載入完成, **When** 使用者按住左鍵拖曳, **Then** 視角繞著場景中心旋轉
2. **Given** 頁面載入完成, **When** 使用者滾動滑鼠滾輪, **Then** 視角放大或縮小
3. **Given** 頁面載入完成, **When** 使用者按住右鍵拖曳, **Then** 視角平移

---

### User Story 4 - 響應式視窗調整 (Priority: P3)

使用者調整瀏覽器視窗大小時，3D 場景能夠自動適應新的視窗尺寸，保持正確的比例和顯示效果。

**Why this priority**: 響應式設計是現代 Web 應用的基本要求，但非核心功能。

**Independent Test**: 調整瀏覽器視窗大小，觀察場景是否正確重新渲染。

**Acceptance Scenarios**:

1. **Given** 頁面正常顯示, **When** 使用者調整視窗大小, **Then** 場景自動調整為新的尺寸
2. **Given** 視窗大小改變, **When** 重新渲染完成, **Then** 文字比例保持正確，不變形

---

### Edge Cases

- 字體檔案載入失敗時，應有適當的錯誤處理或回退機制
- 瀏覽器不支援 WebGL 時，應顯示提示訊息
- 極端視窗尺寸（非常小或非常大）時，場景仍能正常顯示
- 使用者快速連續調整視窗大小時，不應造成效能問題

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 載入壓縮的 JSON 字體檔案（typeface.json.zip）
- **FR-002**: 系統 MUST 使用 Font 類別解析字體並產生文字形狀
- **FR-003**: 系統 MUST 使用 ShapeGeometry 建立文字的填充幾何體
- **FR-004**: 系統 MUST 使用 SVGLoader.pointsToStroke 建立文字的描邊幾何體
- **FR-005**: 系統 MUST 支援三種文字方向：左到右（ltr）、右到左（rtl）、上到下（tb）
- **FR-006**: 系統 MUST 將填充文字設為半透明材質，描邊文字設為實心材質
- **FR-007**: 系統 MUST 正確處理文字中的孔洞（如字母 O、B 等的內部空間）
- **FR-008**: 系統 MUST 將文字置中對齊（基於 bounding box 計算）
- **FR-009**: 系統 MUST 提供 OrbitControls 讓使用者互動瀏覽
- **FR-010**: 系統 MUST 在視窗大小改變時正確調整渲染器和相機

### Key Entities

- **StrokeText**: 描邊文字群組，包含填充網格和描邊網格
- **Font**: 字體資源，用於產生文字形狀
- **Material**: 材質設定，包含實心材質（dark）和半透明材質（lite）
- **Shape**: 文字形狀，包含外輪廓和內部孔洞

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 頁面載入後 3 秒內完成字體載入和文字渲染
- **SC-002**: 三組不同方向的文字同時正確顯示於場景中
- **SC-003**: 使用者能在 60fps 的流暢度下進行視角操作
- **SC-004**: 視窗調整後 100ms 內完成場景重新渲染
- **SC-005**: 文字的填充層和描邊層視覺上正確疊加，無明顯錯位

## Assumptions

- 使用者瀏覽器支援 WebGL 和 ES6 模組
- 字體檔案位於 `fonts/` 目錄下且可正常存取
- 使用者有基本的滑鼠或觸控裝置進行互動
- 網路連線足以載入字體資源（壓縮後約數百 KB）
