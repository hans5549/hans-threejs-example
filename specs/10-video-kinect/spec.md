# Feature Specification: WebGL Video Kinect 深度影片點雲視覺化

**Feature Branch**: `10-video-kinect`  
**Created**: 2025-12-01  
**Status**: Draft  
**Input**: User description: "WebGL Video Kinect - Depth video point cloud visualization using Kinect depth data with custom shaders"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 深度影片點雲視覺化 (Priority: P1)

作為使用者，我希望能夠觀看預錄的 Kinect 深度影片，並以即時 3D 點雲形式呈現，讓我能夠從不同角度觀察深度資料的空間分布。

**Why this priority**: 這是整個功能的核心價值，將 2D 深度影片轉換為 3D 點雲是最基本且最重要的功能。

**Independent Test**: 開啟頁面後，深度影片自動播放，用戶可以看到動態的 3D 點雲在畫面中呈現，點雲的位置根據深度資料計算而來。

**Acceptance Scenarios**:

1. **Given** 頁面載入完成，**When** 深度影片開始播放，**Then** 點雲即時更新，反映影片中的深度資訊
2. **Given** 點雲正在渲染，**When** 影片播放至不同幀，**Then** 點雲形狀隨深度變化而改變
3. **Given** 系統正常運作，**When** 用戶觀看畫面，**Then** 每個點的顏色對應影片中該像素的 RGB 顏色

---

### User Story 2 - 滑鼠互動相機控制 (Priority: P2)

作為使用者，我希望能夠透過滑鼠移動來改變觀看點雲的視角，讓我能夠從不同角度探索 3D 場景。

**Why this priority**: 互動性是提升用戶體驗的關鍵，但在點雲能夠正確渲染之後才有意義。

**Independent Test**: 移動滑鼠時，相機位置平滑跟隨，視角隨之改變，始終朝向點雲中心。

**Acceptance Scenarios**:

1. **Given** 點雲正在顯示，**When** 用戶水平移動滑鼠，**Then** 相機水平位置改變，視角平滑過渡
2. **Given** 點雲正在顯示，**When** 用戶垂直移動滑鼠，**Then** 相機垂直位置改變，視角平滑過渡
3. **Given** 相機正在移動，**When** 停止滑鼠移動，**Then** 相機持續朝向點雲中心點

---

### User Story 3 - GUI 參數即時調整 (Priority: P3)

作為使用者，我希望能夠透過控制面板即時調整深度裁切範圍、點大小和深度偏移等參數，讓我能夠根據需要最佳化視覺效果。

**Why this priority**: 參數調整是進階功能，讓用戶能夠微調顯示效果，但不影響核心功能運作。

**Independent Test**: 調整 GUI 滑桿時，點雲效果立即改變，無需重新載入頁面。

**Acceptance Scenarios**:

1. **Given** GUI 控制面板可見，**When** 用戶調整 nearClipping 值，**Then** 近裁切平面改變，過近的點被隱藏
2. **Given** GUI 控制面板可見，**When** 用戶調整 farClipping 值，**Then** 遠裁切平面改變，過遠的點被隱藏
3. **Given** GUI 控制面板可見，**When** 用戶調整 pointSize 值，**Then** 所有點的大小即時改變
4. **Given** GUI 控制面板可見，**When** 用戶調整 zOffset 值，**Then** 點雲整體在 Z 軸上的位置改變

---

### Edge Cases

- 當影片載入失敗時，系統應顯示錯誤訊息或備用內容
- 當瀏覽器不支援 WebGL 時，應告知用戶使用支援的瀏覽器
- 當視窗大小改變時，渲染畫面應正確調整比例
- 當深度值為極端值（全黑或全白）時，點雲應能正確處理
- 當用戶快速拖動滑鼠時，相機移動應保持平滑不卡頓

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 載入並播放預錄的 Kinect 深度影片（支援 webm 和 mp4 格式）
- **FR-002**: 系統 MUST 使用自訂 Vertex Shader 將影片中每個像素的深度值轉換為 3D 空間座標
- **FR-003**: 系統 MUST 使用 Fragment Shader 為每個點設定對應的顏色和透明度
- **FR-004**: 系統 MUST 建立包含 640x480 個點的 BufferGeometry 點雲結構
- **FR-005**: 系統 MUST 支援透過滑鼠位置控制相機視角
- **FR-006**: 系統 MUST 提供 GUI 控制面板，允許即時調整以下參數：
  - nearClipping (近裁切距離): 1 - 10000
  - farClipping (遠裁切距離): 1 - 10000  
  - pointSize (點大小): 1 - 10
  - zOffset (Z 軸偏移): 0 - 4000
- **FR-007**: 系統 MUST 在視窗大小改變時正確調整渲染區域
- **FR-008**: 系統 MUST 使用加法混合模式 (Additive Blending) 渲染點雲
- **FR-009**: 系統 MUST 使用 VideoTexture 作為 Shader 的輸入貼圖
- **FR-010**: 相機 MUST 始終朝向場景中心點 (0, 0, -1000)

### Key Entities

- **Point Cloud (點雲)**: 由 640x480 = 307,200 個點組成的 3D 幾何體，每個點的位置根據對應像素的深度值計算
- **Video Texture (影片貼圖)**: 從 Kinect 深度影片建立的動態貼圖，提供深度和顏色資訊
- **Shader Material (著色器材質)**: 包含自訂 Vertex 和 Fragment Shader 的材質，用於深度到 3D 座標的轉換
- **Camera (相機)**: 透視投影相機，位置受滑鼠控制，始終注視點雲中心
- **GUI Controller (GUI 控制器)**: lil-gui 控制面板，提供參數調整介面

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 頁面載入後，用戶可以在 3 秒內看到點雲開始渲染
- **SC-002**: 點雲渲染保持流暢，維持至少 30 FPS 的幀率
- **SC-003**: 滑鼠移動與相機反應的延遲不超過 100 毫秒
- **SC-004**: 所有 GUI 參數調整後，視覺效果在下一幀立即更新
- **SC-005**: 視窗調整大小後，渲染在 500 毫秒內完成適應
- **SC-006**: 深度轉換正確呈現 Kinect 視野角度（水平 FOV ~58°、垂直 FOV ~45°）

### Technical Assumptions

- 深度影片解析度為 640x480 像素
- 深度值以 RGB 通道的平均值表示（灰階）
- Kinect 的視野常數：XtoZ = 1.11146, YtoZ = 0.83359
- 預設 nearClipping = 850, farClipping = 4000
- 點雲使用加法混合和透明度 0.2
