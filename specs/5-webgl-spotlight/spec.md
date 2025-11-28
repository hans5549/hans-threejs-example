# Feature Specification: WebGL Spotlight 互動展示

**Feature Branch**: `5-webgl-spotlight`  
**Created**: 2025-11-28  
**Status**: Draft  
**Input**: User description: "實作 Three.js WebGL Spotlight 範例，展示聚光燈效果、動態光源移動、陰影渲染、GUI 控制面板、紋理投射、3D 模型載入與互動式攝影機控制"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 基礎場景與聚光燈展示 (Priority: P1)

使用者開啟網頁後，能夠看到一個包含聚光燈照明的 3D 場景。場景包含一個接收陰影的地面平面和一個投射陰影的 3D 模型，聚光燈會自動環繞場景移動，展示動態光影效果。

**Why this priority**: 這是核心功能，沒有這個基礎場景，其他功能都無法展示。使用者需要先看到視覺效果才能進行互動。

**Independent Test**: 開啟頁面後，無需任何操作即可看到聚光燈照射的 3D 模型和動態移動的光源產生的陰影變化。

**Acceptance Scenarios**:

1. **Given** 使用者開啟範例頁面, **When** 頁面載入完成, **Then** 顯示一個包含地面和 3D 模型的場景，聚光燈產生可見的光照效果
2. **Given** 場景已載入, **When** 動畫執行中, **Then** 聚光燈沿著圓形路徑環繞移動，光影隨之即時變化
3. **Given** 場景已載入, **When** 使用者觀察地面, **Then** 能清楚看到 3D 模型投射的陰影

---

### User Story 2 - 攝影機互動控制 (Priority: P1)

使用者能夠使用滑鼠操控攝影機視角，從不同角度觀察聚光燈效果和陰影細節。

**Why this priority**: 攝影機控制是 3D 展示的基本互動需求，讓使用者能全方位觀察光影效果。

**Independent Test**: 使用滑鼠拖曳、滾輪縮放，確認攝影機能自由旋轉和調整距離。

**Acceptance Scenarios**:

1. **Given** 場景已載入, **When** 使用者按住滑鼠左鍵拖曳, **Then** 攝影機繞著場景中心旋轉
2. **Given** 場景已載入, **When** 使用者滾動滑鼠滾輪, **Then** 攝影機拉近或拉遠（在設定的距離範圍內）
3. **Given** 攝影機移動中, **When** 攝影機嘗試低於地平面, **Then** 攝影機受限制不會穿過地面

---

### User Story 3 - GUI 控制面板調整光源屬性 (Priority: P2)

使用者能夠透過圖形化控制面板即時調整聚光燈的各項屬性，包括顏色、強度、角度、衰減等，觀察不同參數對光照效果的影響。

**Why this priority**: GUI 控制面板是範例的教學價值所在，讓使用者能互動學習光源屬性的效果。

**Independent Test**: 調整 GUI 中的任一參數，確認光照效果即時變化。

**Acceptance Scenarios**:

1. **Given** GUI 面板顯示中, **When** 使用者調整光源顏色, **Then** 聚光燈顏色即時改變
2. **Given** GUI 面板顯示中, **When** 使用者調整光源強度, **Then** 場景亮度即時變化
3. **Given** GUI 面板顯示中, **When** 使用者調整聚光燈角度, **Then** 光照範圍即時擴大或縮小
4. **Given** GUI 面板顯示中, **When** 使用者調整邊緣模糊度（penumbra）, **Then** 光照邊緣的漸變效果即時改變
5. **Given** GUI 面板顯示中, **When** 使用者調整衰減值（decay）, **Then** 光照隨距離衰減的速度即時改變

---

### User Story 4 - 紋理投射功能 (Priority: P2)

使用者能夠透過 GUI 選擇不同的紋理貼圖應用於聚光燈，產生類似投影機的效果。

**Why this priority**: 紋理投射是聚光燈的進階功能展示，增加範例的教學深度。

**Independent Test**: 在 GUI 中切換不同紋理，確認投射效果即時改變。

**Acceptance Scenarios**:

1. **Given** GUI 面板顯示中, **When** 使用者從下拉選單選擇不同紋理, **Then** 聚光燈投射的圖案即時改變
2. **Given** 紋理投射啟用中, **When** 使用者選擇「無」選項, **Then** 聚光燈恢復為普通白光

---

### User Story 5 - 陰影參數調整 (Priority: P3)

使用者能夠透過 GUI 調整陰影相關參數，觀察陰影品質和特性的變化。

**Why this priority**: 陰影參數是進階光照學習內容，適合想深入了解的使用者。

**Independent Test**: 調整陰影焦點和強度參數，確認陰影效果變化。

**Acceptance Scenarios**:

1. **Given** GUI 面板顯示中, **When** 使用者調整陰影焦點（focus）, **Then** 陰影的聚焦程度即時改變
2. **Given** GUI 面板顯示中, **When** 使用者調整陰影強度（shadowIntensity）, **Then** 陰影的深淺程度即時改變

---

### User Story 6 - 輔助視覺化工具 (Priority: P3)

使用者能夠開啟/關閉光源輔助線和陰影攝影機輔助線，幫助理解聚光燈的工作原理。

**Why this priority**: 輔助視覺化是學習工具，對於理解 3D 光照原理很有幫助。

**Independent Test**: 切換 helpers 開關，確認輔助線的顯示與隱藏。

**Acceptance Scenarios**:

1. **Given** GUI 面板顯示中, **When** 使用者勾選 helpers 選項, **Then** 顯示聚光燈輔助線和陰影攝影機範圍框
2. **Given** helpers 已開啟, **When** 使用者取消勾選 helpers, **Then** 輔助線隱藏

---

### Edge Cases

- 當視窗大小改變時，渲染畫面應自動調整比例，不變形
- 當 3D 模型載入失敗時，使用內建幾何體（如球體）作為後備，繼續展示光影效果
- 當紋理載入失敗時，聚光燈使用無紋理的純色光，不影響其他功能
- 當光源強度設為 0 時，場景應只剩環境光照明
- 當聚光燈距離設為 0 時，光照應無距離衰減限制
- 當瀏覽器不支援 WebGL 時，顯示靜態圖片作為降級展示

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 建立包含地面平面的 3D 場景
- **FR-002**: 系統 MUST 載入並顯示 3D 模型（Lucy 雕像）
- **FR-003**: 系統 MUST 建立具有陰影投射能力的聚光燈光源
- **FR-004**: 系統 MUST 實作聚光燈自動環繞移動的動畫
- **FR-005**: 系統 MUST 啟用軟陰影渲染（PCF Soft Shadow）
- **FR-006**: 系統 MUST 提供軌道控制器讓使用者旋轉和縮放視角
- **FR-007**: 系統 MUST 限制攝影機的移動範圍（距離 2-10，不低於地平面）
- **FR-008**: 系統 MUST 提供 GUI 控制面板調整光源屬性
- **FR-009**: 系統 MUST 支援聚光燈紋理投射功能
- **FR-010**: 系統 MUST 支援多種紋理貼圖切換
- **FR-011**: 系統 MUST 提供輔助視覺化工具（光源輔助線、陰影攝影機框線）
- **FR-012**: 系統 MUST 在視窗大小改變時自適應調整渲染
- **FR-013**: 系統 MUST 使用抗鋸齒渲染
- **FR-014**: 系統 MUST 使用中性色調映射（Neutral Tone Mapping）
- **FR-015**: 系統 MUST 在 WebGL 不支援時顯示靜態圖片作為降級展示

### Key Entities

- **場景 (Scene)**: 包含所有 3D 物件、光源和攝影機的容器
- **聚光燈 (SpotLight)**: 主要光源，具有位置、方向、顏色、強度、角度、衰減等屬性
- **3D 模型**: 展示光影效果的主體物件（Lucy 雕像），可投射和接收陰影
- **地面平面**: 接收陰影的表面
- **攝影機**: 使用者觀察場景的視點，具有位置和旋轉控制
- **GUI 控制面板**: 調整光源和陰影參數的使用者介面
- **紋理**: 可應用於聚光燈投射的圖像資源

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 頁面載入後 3 秒內顯示完整的 3D 場景和動態光影效果
- **SC-002**: 攝影機旋轉和縮放操作的響應時間小於 16ms（60fps）
- **SC-003**: GUI 參數調整後，光照效果在 100ms 內反映變化
- **SC-004**: 在主流瀏覽器（Chrome、Firefox、Safari、Edge）中正常運作
- **SC-005**: 視窗調整大小後，畫面在 100ms 內完成自適應
- **SC-006**: 使用者能夠辨識聚光燈投射的紋理圖案
- **SC-007**: 陰影邊緣呈現平滑的軟陰影效果（非硬邊陰影）

## Clarifications

### Session 2025-11-28

- Q: 3D 模型來源策略？ → A: 從 Three.js CDN 載入原始 Lucy 模型
- Q: 紋理資源來源？ → A: 從 Three.js CDN 載入原始紋理
- Q: 外部資源載入失敗處理？ → A: 使用內建幾何體作為後備，繼續展示光影效果
- Q: 行動裝置觸控支援？ → A: 僅支援桌面滑鼠操作，不考慮行動裝置
- Q: WebGL 不支援時的降級處理？ → A: 顯示靜態圖片作為降級展示

## Assumptions

- 使用者的瀏覽器支援 WebGL 2.0
- 使用者的裝置具有基本的 GPU 運算能力
- 網路環境能夠載入外部資源（紋理圖片、3D 模型）
- Three.js 函式庫透過 CDN 或本地引用可用
- 3D 模型（Lucy 雕像）從 Three.js 官方 CDN 載入（外部依賴）
- 紋理貼圖從 Three.js 官方 CDN 載入（外部依賴）

## Constraints

- 僅支援桌面環境滑鼠操作，不支援行動裝置觸控互動
