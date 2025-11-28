# Feature Specification: WebGPU Volumetric Caustics

**Feature Branch**: `4-webgpu-volume-caustics`  
**Created**: 2025-11-28  
**Status**: Draft  
**Input**: User description: "實作 Three.js WebGPU Volume Caustics 範例 - 即時體積焦散效果展示"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 觀看即時體積焦散效果 (Priority: P1)

作為一位訪客，我想要觀看一個透明物體（如玻璃鴨子模型）產生的即時體積焦散光影效果，讓我能夠理解光線如何穿透透明材質並在地面上產生美麗的光斑圖案。

**Why this priority**: 這是本範例的核心視覺體驗，展示 WebGPU 的進階渲染能力，是吸引使用者的主要價值。

**Independent Test**: 頁面載入後可立即看到旋轉的透明鴨子模型，地面上顯示動態焦散光影效果。

**Acceptance Scenarios**:

1. **Given** 使用者開啟頁面，**When** 頁面完成載入，**Then** 顯示一個透明的鴨子模型在聚光燈照射下，地面可見焦散光斑
2. **Given** 頁面已載入，**When** 時間流逝，**Then** 鴨子模型持續旋轉，焦散光影隨之動態變化
3. **Given** 頁面已載入，**When** 觀察焦散效果，**Then** 可見色散效果（chromatic aberration）產生的彩虹邊緣

---

### User Story 2 - 互動式視角控制 (Priority: P1)

作為一位訪客，我想要透過滑鼠或觸控操作來旋轉和縮放視角，以便從不同角度欣賞焦散效果的細節。

**Why this priority**: 互動性是 3D 展示的基本需求，讓使用者能自由探索場景。

**Independent Test**: 使用滑鼠拖曳可旋轉視角，滾輪可縮放距離。

**Acceptance Scenarios**:

1. **Given** 頁面已載入，**When** 使用者拖曳滑鼠左鍵，**Then** 相機視角跟隨旋轉
2. **Given** 頁面已載入，**When** 使用者滾動滑鼠滾輪，**Then** 相機拉近或拉遠
3. **Given** 頁面已載入，**When** 使用者在行動裝置上雙指縮放，**Then** 相機拉近或拉遠

---

### User Story 3 - 體積光效果展示 (Priority: P2)

作為一位訪客，我想要看到霧氣或煙霧中的體積光效果（Volumetric Lighting），讓光束在空間中可見，增強視覺氛圍。

**Why this priority**: 體積光是本範例的進階特效，增添場景的真實感和戲劇效果。

**Independent Test**: 可見光束從聚光燈穿透場景中的煙霧效果。

**Acceptance Scenarios**:

1. **Given** 頁面已載入，**When** 觀察場景，**Then** 可見聚光燈的光束在霧氣中呈現
2. **Given** 頁面已載入，**When** 鴨子旋轉，**Then** 體積光效果隨鴨子位置動態變化

---

### User Story 4 - 響應式視窗調整 (Priority: P3)

作為一位訪客，我想要在調整瀏覽器視窗大小時，畫面能自動適應新的尺寸，保持正確的顯示比例。

**Why this priority**: 響應式設計確保各種螢幕尺寸的使用體驗。

**Independent Test**: 調整視窗大小後畫面自動適應。

**Acceptance Scenarios**:

1. **Given** 頁面已載入，**When** 使用者調整視窗大小，**Then** 3D 畫面自動調整為新的寬高比
2. **Given** 頁面以全螢幕顯示，**When** 使用者退出全螢幕，**Then** 畫面正確縮放

---

### Edge Cases

- **WebGPU 不支援時**：顯示靜態圖片預覽，搭配不支援提示訊息，引導使用者升級瀏覽器
- 模型載入失敗時，頁面應顯示適當的錯誤提示
- **低效能裝置**：不做特別處理，接受可能的低幀率表現（展示範例優先保持視覺完整性）

## Clarifications

### Session 2025-11-28
- Q: 瀏覽器不支援 WebGPU 時應採取什麼策略？ → A: 顯示靜態圖片預覽，搭配不支援提示訊息
- Q: 低效能裝置時系統應如何處理？ → A: 不做特別處理，接受可能的低幀率表現

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 使用 Three.js WebGPU 渲染器（WebGPURenderer）進行場景渲染
- **FR-002**: 系統 MUST 載入並顯示一個 3D 鴨子模型（GLTF 格式）
- **FR-003**: 系統 MUST 對鴨子模型套用透明玻璃材質（MeshPhysicalNodeMaterial with transmission）
- **FR-004**: 系統 MUST 實作焦散效果（Caustics），光線通過透明物體後在地面產生光斑
- **FR-005**: 系統 MUST 實作色散效果（Chromatic Aberration），焦散光斑呈現彩虹色邊緣
- **FR-006**: 系統 MUST 配置聚光燈（SpotLight）作為主要光源並啟用陰影
- **FR-007**: 系統 MUST 實作體積光效果（Volumetric Lighting），光束在霧氣中可見
- **FR-008**: 系統 MUST 實作 3D 噪聲紋理以產生動態煙霧效果
- **FR-009**: 系統 MUST 實作後處理效果，包含 Bloom 效果以增強光暈
- **FR-010**: 系統 MUST 提供 OrbitControls 讓使用者可互動式控制相機視角
- **FR-011**: 系統 MUST 讓鴨子模型持續自動旋轉
- **FR-012**: 系統 MUST 在視窗大小改變時自動調整渲染尺寸和相機比例
- **FR-013**: 系統 MUST 顯示地板平面以接收焦散陰影
- **FR-014**: 系統 MUST 使用 TSL（Three Shading Language）實作自訂焦散著色器

### Key Entities

- **Scene（場景）**: 包含所有 3D 物件的容器
- **Duck Model（鴨子模型）**: GLTF 格式的透明玻璃鴨子，為焦散效果的核心物件
- **SpotLight（聚光燈）**: 產生定向光線和陰影的光源
- **Ground Plane（地板）**: 接收焦散陰影的平面
- **Volumetric Box（體積光盒）**: 用於渲染體積光效果的立方體區域
- **Caustic Map（焦散貼圖）**: 用於投射焦散圖案的紋理
- **3D Noise Texture（3D 噪聲紋理）**: 用於產生動態煙霧效果的紋理

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 頁面在支援 WebGPU 的瀏覽器中能在 5 秒內完成初始載入並顯示完整場景
- **SC-002**: 動畫執行時維持至少 30 FPS 的流暢度（目標 60 FPS）
- **SC-003**: 使用者可透過滑鼠操作在 1 秒內完成視角旋轉回饋
- **SC-004**: 焦散效果清晰可見，包含明顯的色散彩虹效果
- **SC-005**: 體積光效果在場景中可見，增強視覺深度感
- **SC-006**: 視窗調整大小後，畫面在 0.5 秒內完成重新渲染

## Assumptions

- 使用者的瀏覽器支援 WebGPU（Chrome 113+、Edge 113+、或其他支援 WebGPU 的瀏覽器）
- 使用者裝置具備足夠的 GPU 效能來執行即時體積渲染
- 專案結構遵循現有的 `examples/` 目錄組織方式
- 所需的 3D 模型（duck.glb）和紋理檔案可從 Three.js 官方範例取得或本地提供
