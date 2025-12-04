# Feature Specification: WebGL Geometry Spline Editor

**Feature Branch**: `015-spline-editor`  
**Created**: 2025-12-04  
**Status**: Draft  
**Input**: User description: "實作 Three.js WebGL Geometry Spline Editor - 一個互動式 Catmull-Rom 曲線編輯器"
**Reference**: https://threejs.org/examples/#webgl_geometry_spline_editor

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 基本場景與曲線視覺化 (Priority: P1)

使用者開啟範例頁面後，能看到一個完整的 3D 場景，場景中包含地面平面、網格輔助線、預設的控制點以及三種不同顏色的 Catmull-Rom 曲線。這是整個編輯器的基礎視覺化層，所有後續互動功能都依賴於此。

**Why this priority**: 這是 MVP 的核心，沒有視覺化就無法進行任何編輯操作。

**Independent Test**: 可以透過開啟頁面並確認能看到完整場景來驗證，即使沒有任何互動功能，使用者仍能觀察到美觀的 3D 曲線展示。

**Acceptance Scenarios**:

1. **Given** 使用者首次載入頁面, **When** 頁面完成載入, **Then** 顯示包含地面、網格、光源和陰影的 3D 場景
2. **Given** 場景已載入, **When** 檢視曲線, **Then** 應顯示三條不同顏色的曲線（紅色 uniform、綠色 centripetal、藍色 chordal）
3. **Given** 場景已載入, **When** 檢視控制點, **Then** 應顯示 4 個或更多的立方體控制點，每個控制點有隨機顏色

---

### User Story 2 - 相機視角控制 (Priority: P1)

使用者能透過滑鼠控制相機視角，包括軌道旋轉、縮放和平移，以從不同角度觀察曲線。這是 3D 編輯器的基本互動能力。

**Why this priority**: 沒有相機控制，使用者無法從不同角度觀察和編輯曲線，這是基本的 3D 互動需求。

**Independent Test**: 可以透過拖曳滑鼠來旋轉、滾輪縮放來測試相機控制是否正常運作。

**Acceptance Scenarios**:

1. **Given** 場景已顯示, **When** 使用者拖曳滑鼠左鍵, **Then** 相機繞場景中心旋轉
2. **Given** 場景已顯示, **When** 使用者滾動滑鼠滾輪, **Then** 相機縮放靠近或遠離場景
3. **Given** 場景已顯示, **When** 使用者拖曳滑鼠右鍵, **Then** 相機平移視角

---

### User Story 3 - 控制點拖曳編輯 (Priority: P1)

使用者能點擊選取場景中的控制點，並透過 TransformControls 拖曳移動控制點位置，曲線會即時更新以反映控制點的新位置。

**Why this priority**: 這是編輯器的核心功能，是使用者與曲線互動的主要方式。

**Independent Test**: 可以透過點擊控制點、拖曳移動並觀察曲線變化來獨立測試。

**Acceptance Scenarios**:

1. **Given** 場景中有控制點, **When** 使用者點擊一個控制點, **Then** 該控制點顯示 TransformControls 操作手柄
2. **Given** 控制點已被選取, **When** 使用者拖曳 TransformControls, **Then** 控制點跟隨移動
3. **Given** 控制點正在被拖曳, **When** 控制點位置改變, **Then** 所有曲線即時更新以通過新位置
4. **Given** 正在拖曳控制點, **When** 使用者嘗試旋轉相機, **Then** 軌道控制暫時停用以避免衝突

---

### User Story 4 - 新增控制點 (Priority: P2)

使用者能透過 GUI 按鈕新增新的控制點到曲線上，新增的控制點會出現在隨機位置，曲線會更新以包含新的控制點。

**Why this priority**: 擴展曲線是重要的編輯功能，但基本視覺化和拖曳編輯優先。

**Independent Test**: 可以透過點擊「Add Point」按鈕並觀察新控制點出現來測試。

**Acceptance Scenarios**:

1. **Given** 曲線有 N 個控制點, **When** 使用者點擊「Add Point」按鈕, **Then** 新控制點出現在場景中
2. **Given** 新控制點已加入, **When** 查看曲線, **Then** 曲線更新為通過 N+1 個控制點
3. **Given** 新控制點已加入, **When** 使用者點擊新控制點, **Then** 可以像其他控制點一樣選取和拖曳

---

### User Story 5 - 移除控制點 (Priority: P2)

使用者能透過 GUI 按鈕移除最後一個控制點，但系統會保持至少 4 個控制點以確保曲線的完整性。

**Why this priority**: 與新增功能配對，提供完整的控制點管理能力。

**Independent Test**: 可以透過點擊「Remove Point」按鈕並觀察控制點減少來測試。

**Acceptance Scenarios**:

1. **Given** 曲線有 5 個或更多控制點, **When** 使用者點擊「Remove Point」按鈕, **Then** 最後一個控制點被移除
2. **Given** 曲線正好有 4 個控制點, **When** 使用者點擊「Remove Point」按鈕, **Then** 不執行任何操作（保持最少 4 點）
3. **Given** 被移除的控制點正被選取, **When** 控制點被移除, **Then** TransformControls 自動脫離

---

### User Story 6 - 曲線類型切換 (Priority: P2)

使用者能透過 GUI 控制面板的勾選框來開關不同類型的曲線顯示（uniform、centripetal、chordal），並調整 uniform 曲線的張力參數。

**Why this priority**: 提供曲線參數調整能力，增強編輯器的功能性。

**Independent Test**: 可以透過勾選/取消勾選 GUI 選項並觀察曲線顯示變化來測試。

**Acceptance Scenarios**:

1. **Given** 三條曲線都在顯示, **When** 使用者取消勾選「uniform」, **Then** 紅色曲線隱藏
2. **Given** 三條曲線都在顯示, **When** 使用者取消勾選「centripetal」, **Then** 綠色曲線隱藏
3. **Given** 三條曲線都在顯示, **When** 使用者取消勾選「chordal」, **Then** 藍色曲線隱藏
4. **Given** uniform 曲線在顯示, **When** 使用者調整「tension」滑桿, **Then** 紅色曲線形狀隨張力值變化

---

### User Story 7 - 匯出曲線資料 (Priority: P3)

使用者能透過 GUI 按鈕將目前曲線的控制點座標匯出到瀏覽器主控台，方便開發者取得曲線資料用於其他用途。

**Why this priority**: 這是進階功能，對於開發者整合有用但非核心編輯體驗。

**Independent Test**: 可以透過點擊「Export Spline」按鈕並檢查瀏覽器主控台輸出來測試。

**Acceptance Scenarios**:

1. **Given** 場景中有控制點, **When** 使用者點擊「Export」按鈕, **Then** 控制點座標以 JavaScript 陣列格式輸出到主控台
2. **Given** 使用者已修改控制點位置, **When** 匯出曲線, **Then** 匯出的座標反映當前修改後的位置

---

### Edge Cases

- **視窗大小改變**: 當視窗大小改變時，場景應正確調整並保持比例（監聽 resize 事件並更新相機和渲染器尺寸）
- **快速連續操作**: 當使用者快速連續點擊「Add Point」時，每次點擊都正常新增控制點
- **最少控制點保護**: 當使用者嘗試移除控制點至少於 4 個時，忽略移除操作以保持曲線完整性
- **空點擊處理**: 當滑鼠點擊位置沒有控制點時，不選取任何控制點，TransformControls 不顯示

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 渲染一個包含地面平面、網格輔助線和光源的 3D 場景
- **FR-002**: 系統 MUST 顯示三條 Catmull-Rom 曲線（uniform/centripetal/chordal），每條使用不同顏色
- **FR-003**: 系統 MUST 以立方體 Mesh 形式顯示可拖曳的控制點
- **FR-004**: 系統 MUST 支援 OrbitControls 讓使用者旋轉、縮放、平移相機視角
- **FR-005**: 系統 MUST 支援透過 Raycaster 點擊選取控制點
- **FR-006**: 系統 MUST 使用 TransformControls 讓使用者拖曳移動選取的控制點
- **FR-007**: 系統 MUST 在控制點位置改變時即時更新所有曲線
- **FR-008**: 系統 MUST 提供 GUI 面板，包含曲線類型開關、張力調整、新增/移除控制點按鈕、匯出按鈕
- **FR-009**: 系統 MUST 支援透過 GUI 新增新控制點到場景
- **FR-010**: 系統 MUST 支援透過 GUI 移除最後一個控制點（保持最少 4 點）
- **FR-011**: 系統 MUST 支援切換顯示/隱藏各類型曲線
- **FR-012**: 系統 MUST 支援調整 uniform 曲線的張力參數（0-1 範圍）
- **FR-013**: 系統 MUST 支援將控制點座標匯出到瀏覽器主控台
- **FR-014**: 系統 MUST 在拖曳控制點時暫時停用軌道控制以避免衝突
- **FR-015**: 系統 MUST 支援視窗大小調整時自動更新渲染尺寸

### Key Entities

- **控制點 (Control Point)**: 曲線上的可編輯節點，以立方體 Mesh 表示，具有 3D 位置座標
- **曲線 (Spline)**: 由控制點定義的 Catmull-Rom 曲線，有三種類型（uniform、centripetal、chordal）
- **場景 (Scene)**: 3D 環境容器，包含地面、網格、光源、控制點和曲線
- **TransformControls**: 控制點編輯工具，提供拖曳操作手柄

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 頁面載入後 3 秒內完成場景渲染，使用者能看到完整的 3D 場景
- **SC-002**: 使用者能在 5 秒內完成選取控制點並開始拖曳的操作流程
- **SC-003**: 控制點拖曳時，曲線更新延遲不超過 100 毫秒，使用者感知為即時回饋
- **SC-004**: 相機操作（旋轉、縮放、平移）流暢，維持 30fps 以上的畫面更新率
- **SC-005**: 所有 GUI 控制項（曲線開關、張力滑桿、按鈕）能在 1 秒內完成操作回饋
- **SC-006**: 系統能支援至少 20 個控制點而不影響操作流暢度
- **SC-007**: 視窗大小調整後 500 毫秒內完成場景重新渲染

## Assumptions

- 使用者使用現代瀏覽器（支援 WebGL 2.0）
- 使用者設備具備基本 GPU 加速能力
- Three.js 函式庫及相關 addons（OrbitControls、TransformControls、lil-gui）可從 CDN 或本地取得
- 預設載入 4 個控制點作為初始曲線
- 控制點的初始位置參考原始範例中的預設座標
