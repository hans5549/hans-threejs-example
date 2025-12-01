# Feature Specification: Particle Earth Sphere

**Feature Branch**: `14-particle-earth`  
**Created**: 2025-12-01  
**Status**: Draft  
**Input**: 建立一個地球樣式的球體，球體表面為粒子狀態，參考 webgl_video_kinect 範例

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 檢視粒子地球 (Priority: P1)

作為一個網頁訪客，我希望能夠看到一個由粒子組成的地球球體，讓我可以欣賞這種獨特的視覺效果。

**Why this priority**: 這是核心功能，沒有這個功能就無法展示粒子地球的視覺效果。

**Independent Test**: 開啟網頁後立即可見粒子組成的地球球體，且球體正在自動旋轉。

**Acceptance Scenarios**:

1. **Given** 使用者載入網頁，**When** 頁面完成載入，**Then** 可見一個由粒子組成的地球球體顯示在畫面中央
2. **Given** 地球球體已顯示，**When** 時間經過，**Then** 地球球體會持續自動旋轉
3. **Given** 粒子地球正在顯示，**When** 檢視球體表面，**Then** 可以看到粒子呈現地球的陸地與海洋分布

---

### User Story 2 - 滑鼠互動旋轉 (Priority: P2)

作為一個網頁訪客，我希望能夠透過滑鼠移動來改變觀看地球的角度，讓我可以從不同方向欣賞粒子地球。

**Why this priority**: 互動性是增加使用者體驗的重要元素，讓使用者可以主動探索。

**Independent Test**: 移動滑鼠時，攝影機視角會跟隨滑鼠位置平滑移動。

**Acceptance Scenarios**:

1. **Given** 粒子地球已顯示，**When** 使用者將滑鼠向左移動，**Then** 攝影機視角平滑地向左旋轉
2. **Given** 粒子地球已顯示，**When** 使用者將滑鼠向右移動，**Then** 攝影機視角平滑地向右旋轉
3. **Given** 粒子地球已顯示，**When** 使用者將滑鼠向上或向下移動，**Then** 攝影機視角相應地上下調整

---

### User Story 3 - 視覺效果調整 (Priority: P3)

作為一個網頁訪客，我希望能夠調整粒子的顯示效果，讓我可以根據個人喜好客製化視覺呈現。

**Why this priority**: 這是進階功能，提供更多控制選項但非核心必要。

**Independent Test**: 透過控制面板調整粒子大小，可以即時看到粒子尺寸變化。

**Acceptance Scenarios**:

1. **Given** 粒子地球已顯示，**When** 使用者調整粒子大小滑桿，**Then** 畫面中的粒子尺寸即時更新
2. **Given** 粒子地球已顯示，**When** 使用者調整旋轉速度，**Then** 地球自動旋轉的速度相應改變

---

### Edge Cases

- 當瀏覽器不支援 WebGL 時會發生什麼？應顯示友善的錯誤訊息
- 當視窗大小改變時，畫面應該如何調整？應自動調整渲染尺寸以適應新視窗大小
- 當滑鼠移出視窗時，攝影機應該如何行為？應維持最後的位置或緩慢回到預設位置

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 顯示一個由粒子組成的球體，呈現地球的外觀
- **FR-002**: 系統 MUST 使用地球紋理貼圖來決定每個粒子的顏色
- **FR-003**: 球體 MUST 自動持續旋轉，模擬地球自轉
- **FR-004**: 系統 MUST 回應滑鼠移動，平滑地調整攝影機視角
- **FR-005**: 系統 MUST 在視窗大小改變時自動調整渲染尺寸
- **FR-006**: 系統 MUST 使用 Three.js 的 Points 幾何體來渲染粒子
- **FR-007**: 系統 MUST 使用自訂 Shader 來處理粒子的位置與顏色
- **FR-008**: 系統 SHOULD 提供控制面板讓使用者調整粒子大小
- **FR-009**: 系統 SHOULD 提供控制面板讓使用者調整旋轉速度
- **FR-010**: 粒子 MUST 使用加法混合模式以產生發光效果

### Key Entities

- **Particle Earth（粒子地球）**: 核心視覺物件，由數萬個粒子組成的球體，每個粒子位置對應球面座標，顏色來自地球紋理貼圖
- **Camera（攝影機）**: 透視攝影機，會跟隨滑鼠移動平滑調整位置
- **Shader Material（著色器材質）**: 自訂的頂點和片段著色器，控制粒子的位置、大小和顏色
- **Earth Texture（地球紋理）**: 地球表面的圖片紋理，用於決定每個粒子的顏色

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 頁面載入後 3 秒內完成粒子地球的初始渲染並開始顯示
- **SC-002**: 粒子數量達到足夠密度（至少 10,000 個粒子）以清晰呈現地球輪廓
- **SC-003**: 攝影機跟隨滑鼠移動的延遲感覺流暢自然（使用緩動效果）
- **SC-004**: 動畫在主流瀏覽器上維持 60 FPS 的流暢度
- **SC-005**: 視窗大小改變後 100 毫秒內完成畫面重新調整

## Assumptions

- 使用者的瀏覽器支援 WebGL
- 使用者裝置具備足夠的 GPU 效能來渲染粒子效果
- 地球紋理貼圖為標準等距柱狀投影（Equirectangular projection）格式
- 參考 webgl_video_kinect 範例的粒子渲染技術，但將影片深度圖替換為球面座標計算
