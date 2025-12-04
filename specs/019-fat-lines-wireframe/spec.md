# Feature Specification: Fat Lines Wireframe

**Feature Branch**: `019-fat-lines-wireframe`  
**Created**: 2025-12-04  
**Status**: Draft  
**Input**: User description: "實作 Three.js Fat Lines Wireframe 範例，包含可變線寬、虛線支援、雙視口比較和互動式 GUI 控制"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 檢視 Fat Lines Wireframe 3D 模型 (Priority: P1)

使用者開啟頁面後，能夠在畫面上看到一個使用「粗線條」(Fat Lines) 技術渲染的 Icosahedron (二十面體) 線框模型，線條具有可見的寬度，而非傳統 WebGL 的 1 像素細線。

**Why this priority**: 這是此功能的核心展示目的 - 展示 Fat Lines 技術如何提供比標準 WebGL 線條更豐富的視覺效果。

**Independent Test**: 開啟頁面即可看到藍色粗線條渲染的 3D 線框模型，線條寬度明顯大於 1 像素。

**Acceptance Scenarios**:

1. **Given** 使用者開啟頁面, **When** 頁面載入完成, **Then** 顯示一個藍色粗線條渲染的 Icosahedron 線框模型
2. **Given** 3D 模型已顯示, **When** 使用者觀察線條, **Then** 線條寬度為 5 像素（預設值），明顯比傳統 1 像素線條粗

---

### User Story 2 - 使用軌道控制旋轉與縮放模型 (Priority: P1)

使用者能夠透過滑鼠拖曳來旋轉 3D 模型視角，並使用滾輪來縮放視圖，以從不同角度檢視 Fat Lines 的渲染效果。

**Why this priority**: 互動式 3D 檢視是展示技術的基本需求，讓使用者能從各角度觀察線條效果。

**Independent Test**: 使用滑鼠拖曳可旋轉模型，滾輪可縮放視圖，縮放範圍在 10 到 500 之間。

**Acceptance Scenarios**:

1. **Given** 3D 模型已顯示, **When** 使用者按住滑鼠左鍵並拖曳, **Then** 模型視角隨之旋轉
2. **Given** 3D 模型已顯示, **When** 使用者滾動滑鼠滾輪, **Then** 視圖放大或縮小
3. **Given** 視圖縮放中, **When** 縮放距離達到限制, **Then** 縮放停止在最小距離 10 或最大距離 500

---

### User Story 3 - 調整線條寬度 (Priority: P2)

使用者能夠透過 GUI 控制面板即時調整線條寬度，從 1 像素到 10 像素之間變化，觀察不同寬度的視覺效果。

**Why this priority**: 可調整線寬是 Fat Lines 技術的核心優勢，展示其相較於標準 WebGL 線條的靈活性。

**Independent Test**: 拖曳線寬滑桿可即時看到線條粗細變化。

**Acceptance Scenarios**:

1. **Given** GUI 面板已顯示, **When** 使用者拖曳 "width (px)" 滑桿, **Then** 模型線條寬度即時更新
2. **Given** 線寬滑桿, **When** 使用者將滑桿拖至最小值 1, **Then** 線條變為最細
3. **Given** 線寬滑桿, **When** 使用者將滑桿拖至最大值 10, **Then** 線條變為最粗

---

### User Story 4 - 切換虛線模式 (Priority: P2)

使用者能夠透過 GUI 開關啟用或停用虛線效果，並可進一步調整虛線的縮放比例和間隔比例。

**Why this priority**: 虛線支援是 Fat Lines 的進階功能，展示其多樣的樣式選項。

**Independent Test**: 切換虛線開關可看到線條在實線與虛線之間變化。

**Acceptance Scenarios**:

1. **Given** GUI 面板已顯示, **When** 使用者勾選 "dashed" 選項, **Then** 線條從實線變為虛線
2. **Given** 虛線模式已啟用, **When** 使用者調整 "dash scale" 滑桿, **Then** 虛線的縮放比例即時變化
3. **Given** 虛線模式已啟用, **When** 使用者選擇不同的 "dash / gap" 比例, **Then** 虛線與間隔的比例隨之改變

---

### User Story 5 - 比較 Fat Lines 與標準線條 (Priority: P2)

使用者能夠在 Fat Lines 渲染和標準 WebGL LineSegments 渲染之間切換，以比較兩種技術的差異。

**Why this priority**: 對比功能讓使用者直觀理解 Fat Lines 技術的優勢。

**Independent Test**: 使用類型選擇器切換，可看到渲染方式在 Fat Lines 和標準線條之間變化。

**Acceptance Scenarios**:

1. **Given** GUI 面板已顯示, **When** 使用者選擇 "Line Type" 為 0 (LineGeometry), **Then** 顯示 Fat Lines 渲染的模型
2. **Given** GUI 面板已顯示, **When** 使用者選擇 "Line Type" 為 1 (gl.LINE), **Then** 顯示標準 WebGL 線條渲染的模型

---

### User Story 6 - 雙視口顯示（插入視口） (Priority: P3)

畫面右下角顯示一個小型插入視口 (Inset Viewport)，從第二個攝影機角度同步顯示相同的模型，提供額外的視角參考。

**Why this priority**: 雙視口是進階視覺化功能，增強使用者對 3D 空間的理解。

**Independent Test**: 頁面載入後，右下角出現一個正方形的小視口，顯示與主視口相同的模型但可能有不同視角。

**Acceptance Scenarios**:

1. **Given** 頁面載入完成, **When** 使用者檢視畫面, **Then** 右下角顯示一個小型插入視口
2. **Given** 插入視口已顯示, **When** 使用者旋轉主視口, **Then** 插入視口中的模型也同步更新
3. **Given** 瀏覽器視窗大小改變, **When** resize 事件觸發, **Then** 插入視口大小按比例調整（高度的 1/4）

---

### Edge Cases

- 當瀏覽器視窗縮小到極端尺寸時，插入視口應維持正方形比例並保持可見
- 當線寬設為 1 像素時，Fat Lines 仍應正確渲染（不會消失）
- 在切換線條類型時，虛線設定應在兩種渲染模式間同步
- 當 dash scale 設為極端值（0.5 或 1）時，虛線效果應保持正確顯示

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 使用 WireframeGeometry2 和 LineMaterial 建立 Fat Lines 線框模型
- **FR-002**: 系統 MUST 使用 Icosahedron 幾何體（半徑 20，細分層級 1）作為展示模型
- **FR-003**: 系統 MUST 提供 OrbitControls 軌道控制功能，支援滑鼠拖曳旋轉和滾輪縮放
- **FR-004**: 系統 MUST 限制軌道控制的縮放距離在 10 到 500 之間
- **FR-005**: 系統 MUST 提供 GUI 控制面板，包含線條類型、線寬、虛線開關等控制項
- **FR-006**: 系統 MUST 支援線寬調整範圍為 1 到 10 像素
- **FR-007**: 系統 MUST 支援虛線模式，包含 dash scale（0.5 到 1）和 dash/gap 比例選項（2:1、1:1、1:2）
- **FR-008**: 系統 MUST 支援在 Fat Lines (Wireframe) 和標準 WebGL (LineSegments) 之間切換
- **FR-009**: 系統 MUST 在右下角顯示插入視口，尺寸為視窗高度的 1/4
- **FR-010**: 系統 MUST 支援視窗大小調整，自動更新攝影機比例和視口大小
- **FR-011**: 系統 MUST 使用透明背景（clearColor alpha 為 0）
- **FR-012**: 系統 MUST 顯示 FPS 統計資訊 (Stats)

### Key Entities

- **Wireframe**: Fat Lines 版本的線框模型，使用 WireframeGeometry2 + LineMaterial
- **LineSegments**: 標準 WebGL 版本的線框模型，使用 WireframeGeometry + LineBasicMaterial/LineDashedMaterial
- **Camera (主攝影機)**: 透視攝影機，FOV 40 度，控制主視口
- **Camera2 (副攝影機)**: 透視攝影機，FOV 40 度，控制插入視口
- **LineMaterial**: Fat Lines 材質，支援可變線寬和虛線效果
- **GUI Parameters**: 控制參數物件，包含 lineType、width、dashed、dashScale、dashGap

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 使用者能夠在 3 秒內看到完整渲染的 3D 線框模型
- **SC-002**: 線寬調整從 1 到 10 像素的視覺變化明顯可辨識
- **SC-003**: 虛線與實線的切換在 100 毫秒內完成視覺更新
- **SC-004**: 軌道控制的旋轉和縮放操作流暢，無明顯延遲
- **SC-005**: 雙視口同步顯示，無明顯的視覺不一致
- **SC-006**: 視窗大小調整後，所有 UI 元素在 500 毫秒內正確重新排列
- **SC-007**: 頁面在主流瀏覽器（Chrome、Firefox、Edge）中正確運行

## Assumptions

- 使用者的瀏覽器支援 WebGL
- 使用者設備具備足夠的 GPU 效能來渲染 3D 圖形
- 專案使用 ES modules 和 import maps 載入 Three.js
- Three.js addons（LineMaterial、Wireframe、WireframeGeometry2）可從 CDN 或本地取得
- lil-gui 函式庫用於 GUI 控制面板
- Stats.js 函式庫用於 FPS 顯示
