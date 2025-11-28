# Feature Specification: Unreal Bloom Post-processing Effect

**Feature Branch**: `2-unreal-bloom`  
**Created**: 2025-11-28  
**Status**: Draft  
**Input**: User description: "實作 Three.js Unreal Bloom 後處理效果，參考官方範例 webgl_postprocessing_unreal_bloom"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 查看 Bloom 發光效果的 3D 模型 (Priority: P1)

使用者開啟網頁時，能看到一個具有發光效果的動畫 3D 模型（Primary Ion Drive 太空引擎）。模型會自動播放動畫，並且散發出柔和的光暈效果，營造出科幻氛圍。

**Why this priority**: 這是核心視覺體驗，沒有 Bloom 效果就失去了本功能的意義。使用者必須能立即看到發光效果才能驗證功能是否正常運作。

**Independent Test**: 開啟網頁後，視覺上確認 3D 模型周圍有明顯的光暈效果，且模型動畫正在播放。

**Acceptance Scenarios**:

1. **Given** 使用者首次載入頁面, **When** 頁面完成載入, **Then** 顯示帶有 Bloom 發光效果的動畫 3D 模型
2. **Given** 頁面已載入, **When** 動畫播放中, **Then** 發光效果持續應用於模型的亮部區域
3. **Given** 頁面已載入, **When** 使用者等待, **Then** 模型動畫持續循環播放

---

### User Story 2 - 使用滑鼠旋轉視角觀察模型 (Priority: P1)

使用者可以使用滑鼠拖曳來旋轉相機視角，從不同角度觀察 3D 模型和 Bloom 效果。

**Why this priority**: 互動性是 3D 展示的核心體驗，使用者需要能自由探索模型才能完整欣賞 Bloom 效果。

**Independent Test**: 使用滑鼠在畫面上拖曳，確認視角會隨之旋轉。

**Acceptance Scenarios**:

1. **Given** 頁面已載入, **When** 使用者按住滑鼠左鍵並拖曳, **Then** 相機視角繞著模型旋轉
2. **Given** 頁面已載入, **When** 使用者滾動滑鼠滾輪, **Then** 相機縮放（有最近和最遠限制）
3. **Given** 頁面已載入, **When** 使用者嘗試旋轉超過垂直限制, **Then** 相機停止在最大角度不會翻轉

---

### User Story 3 - 即時調整 Bloom 效果參數 (Priority: P2)

使用者可以透過 GUI 控制面板即時調整 Bloom 效果的各項參數，包括閾值、強度、半徑和曝光度，並立即看到效果變化。

**Why this priority**: 參數調整讓使用者能客製化視覺效果，增加互動性和學習價值，但基本展示不需要此功能也能運作。

**Independent Test**: 調整 GUI 中任一滑桿，確認畫面效果立即更新。

**Acceptance Scenarios**:

1. **Given** 頁面已載入且 GUI 可見, **When** 使用者調整 threshold 滑桿, **Then** Bloom 效果的亮度閾值即時改變
2. **Given** 頁面已載入且 GUI 可見, **When** 使用者調整 strength 滑桿, **Then** Bloom 效果的強度即時改變
3. **Given** 頁面已載入且 GUI 可見, **When** 使用者調整 radius 滑桿, **Then** Bloom 效果的擴散半徑即時改變
4. **Given** 頁面已載入且 GUI 可見, **When** 使用者調整 exposure 滑桿, **Then** 整體畫面曝光度即時改變

---

### User Story 4 - 響應式視窗調整 (Priority: P3)

使用者調整瀏覽器視窗大小時，3D 場景能自動適應新的尺寸，保持正確的畫面比例和完整的視覺效果。

**Why this priority**: 響應式設計確保不同螢幕尺寸的使用者都能正常使用，但不影響核心功能。

**Independent Test**: 調整瀏覽器視窗大小，確認畫面比例正確且 Bloom 效果仍然正常。

**Acceptance Scenarios**:

1. **Given** 頁面已載入, **When** 使用者調整瀏覽器視窗大小, **Then** 3D 場景自動調整為新的尺寸且比例正確
2. **Given** 頁面已載入, **When** 視窗尺寸改變, **Then** Bloom 效果繼續正常渲染

---

### Edge Cases

- 當使用者快速連續調整 GUI 參數時，系統應保持流暢不卡頓
- 當視窗尺寸非常小（如 200x200）時，場景仍應正常渲染
- 當 3D 模型載入失敗時，應有適當的錯誤處理
- 當瀏覽器不支援 WebGL 時，應顯示友善的錯誤訊息

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 載入並顯示 GLTF 格式的 3D 模型（Primary Ion Drive）
- **FR-002**: 系統 MUST 播放 3D 模型內建的動畫
- **FR-003**: 系統 MUST 應用 Unreal Bloom 後處理效果於渲染輸出
- **FR-004**: 系統 MUST 提供 OrbitControls 讓使用者能旋轉和縮放視角
- **FR-005**: 系統 MUST 提供 GUI 控制面板來調整 Bloom 參數
- **FR-006**: 系統 MUST 支援調整 Bloom threshold（閾值）參數，範圍 0.0 至 1.0
- **FR-007**: 系統 MUST 支援調整 Bloom strength（強度）參數，範圍 0.0 至 3.0
- **FR-008**: 系統 MUST 支援調整 Bloom radius（半徑）參數，範圍 0.0 至 1.0
- **FR-009**: 系統 MUST 支援調整曝光度（exposure）參數，範圍 0.1 至 2.0
- **FR-010**: 系統 MUST 在視窗大小改變時自動調整渲染尺寸
- **FR-011**: 系統 MUST 顯示效能統計資訊（FPS 計數器）
- **FR-012**: 系統 MUST 限制相機視角的垂直旋轉範圍（不超過 90 度）
- **FR-013**: 系統 MUST 限制相機縮放的距離範圍（最近 3 單位，最遠 8 單位）

### Key Entities

- **Scene（場景）**: 包含所有 3D 物件、燈光的容器
- **Camera（相機）**: 透視相機，決定使用者的觀察視角
- **3D Model（模型）**: Primary Ion Drive GLTF 模型，包含網格和動畫
- **Lights（燈光）**: 環境光和點光源，提供場景照明
- **Effect Composer（效果合成器）**: 管理後處理效果的渲染流程
- **Bloom Pass（Bloom 通道）**: 負責產生發光效果的後處理步驟
- **GUI Controls（控制介面）**: 參數調整的使用者介面

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 頁面載入完成後 3 秒內，使用者能看到完整的 Bloom 發光效果
- **SC-002**: 調整任一 GUI 參數後，畫面在 100 毫秒內反映變化
- **SC-003**: 場景維持至少 30 FPS 的流暢渲染（一般硬體環境）
- **SC-004**: 視窗大小改變後 500 毫秒內完成重新渲染
- **SC-005**: 相機控制響應靈敏，滑鼠操作與視角變化同步
- **SC-006**: 100% 的使用者能在首次使用時成功調整 Bloom 效果參數

## Assumptions

- 使用者的瀏覽器支援 WebGL
- 使用者有基本的滑鼠操作能力
- 網路環境允許載入外部 3D 模型檔案（約 1MB）
- 使用者的設備效能足以執行即時 3D 渲染
