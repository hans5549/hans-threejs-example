# Feature Specification: WebGL Custom Attributes Lines

**Feature Branch**: `12-custom-attributes-lines`  
**Created**: 2025-12-01  
**Status**: Draft  
**Input**: User description: "實作 Three.js WebGL Custom Attributes Lines 範例 - 使用自訂 shader attributes 建立動態線條文字效果，展示 TextGeometry 結合 ShaderMaterial 的進階 WebGL 渲染技術"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 檢視動態線條文字效果 (Priority: P1)

使用者開啟網頁後，可以看到一個「three.js」的 3D 文字以線條方式渲染，文字會自動旋轉並產生動態位移效果，線條會隨時間呈現波動和顏色變化。

**Why this priority**: 這是核心展示功能，展示了 ShaderMaterial 結合自訂 attributes 的主要視覺效果。

**Independent Test**: 開啟網頁後應立即看到動態渲染的 3D 文字線條效果。

**Acceptance Scenarios**:

1. **Given** 使用者開啟範例網頁, **When** 頁面載入完成, **Then** 應顯示「three.js」3D 文字以線條形式渲染
2. **Given** 頁面正常顯示, **When** 時間經過, **Then** 文字應自動旋轉且線條產生波動位移效果
3. **Given** 動畫播放中, **When** 觀察線條顏色, **Then** 線條顏色應呈現漸變效果（HSL 色彩遍歷）

---

### User Story 2 - 自適應視窗大小 (Priority: P2)

使用者調整瀏覽器視窗大小時，3D 場景會自動調整以適應新的視窗尺寸，保持正確的比例和顯示效果。

**Why this priority**: 確保良好的使用者體驗，支援不同螢幕尺寸。

**Independent Test**: 調整瀏覽器視窗大小，場景應正確重新渲染。

**Acceptance Scenarios**:

1. **Given** 範例正在執行, **When** 使用者調整視窗大小, **Then** 渲染畫面應自動調整以適應新的視窗尺寸
2. **Given** 視窗大小改變, **When** 渲染更新, **Then** 3D 物件應保持正確的長寬比

---

### User Story 3 - 效能監控顯示 (Priority: P3)

使用者可以在畫面上看到即時的效能監控資訊（FPS），以了解動畫的執行效能。

**Why this priority**: 輔助功能，幫助使用者和開發者了解渲染效能。

**Independent Test**: 開啟頁面後應在左上角看到 Stats 效能監控面板。

**Acceptance Scenarios**:

1. **Given** 範例正在執行, **When** 查看畫面左上角, **Then** 應顯示 Stats 效能監控面板
2. **Given** Stats 面板顯示中, **When** 動畫執行, **Then** FPS 數值應即時更新

---

### Edge Cases

- 字型檔案載入失敗時，使用備用的簡單幾何圖形（如 BoxGeometry）替代文字繼續展示動畫效果
- 瀏覽器不支援 WebGL 時，應顯示適當的提示訊息
- 極端視窗尺寸（非常小或非常大）時，場景仍應正確渲染

## Clarifications

### Session 2025-12-01

- Q: 當字型檔案載入失敗時，系統應該如何處理？ → A: 使用備用的簡單幾何圖形（如方塊）替代文字繼續展示

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 載入 Helvetiker Bold 字型檔案以建立 TextGeometry
- **FR-002**: 系統 MUST 使用 ShaderMaterial 配合自訂 vertex shader 和 fragment shader
- **FR-003**: 系統 MUST 定義 `displacement` 和 `customColor` 兩個自訂 buffer attributes
- **FR-004**: 系統 MUST 定義 `amplitude`、`opacity` 和 `color` 三個 uniforms
- **FR-005**: 系統 MUST 使用 THREE.Line 而非 Mesh 來渲染 TextGeometry
- **FR-006**: 系統 MUST 實作 Additive Blending 透明效果
- **FR-007**: 系統 MUST 實作動畫迴圈，更新 amplitude、displacement 和 color uniforms
- **FR-008**: 系統 MUST 監聽視窗 resize 事件並更新相機和渲染器
- **FR-009**: 系統 MUST 整合 Stats.js 顯示效能監控

### Key Entities

- **TextGeometry**: 使用字型建立的 3D 文字幾何體，包含 position 頂點資料
- **ShaderMaterial**: 自訂著色器材質，包含 uniforms 和 vertex/fragment shaders
- **Buffer Attributes**: 
  - `displacement`: 每個頂點的位移向量 (vec3)
  - `customColor`: 每個頂點的自訂顏色 (vec3)
- **Uniforms**:
  - `amplitude`: 位移振幅係數 (float)
  - `opacity`: 線條透明度 (float)
  - `color`: 基礎顏色 (vec3)

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 頁面載入後 3 秒內完成字型載入並顯示 3D 文字
- **SC-002**: 動畫效果在主流瀏覽器（Chrome、Firefox、Safari、Edge）維持 30+ FPS
- **SC-003**: 視窗調整大小後 100ms 內完成重新渲染
- **SC-004**: 線條顏色在動畫循環中呈現完整的 HSL 色相遍歷
- **SC-005**: 100% 的使用者可以在支援 WebGL 的瀏覽器中看到完整效果
