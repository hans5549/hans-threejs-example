# Feature Specification: BufferGeometry DrawRange 互動式粒子網絡

**Feature Branch**: `7-buffergeometry-drawrange`  
**Created**: 2025-11-28  
**Status**: Draft  
**Input**: 實作 Three.js BufferGeometry DrawRange 範例 - 一個互動式 3D 粒子網絡視覺化展示

## 概述

本功能實作一個互動式 3D 粒子網絡視覺化系統，展示 Three.js 的 BufferGeometry 和 drawRange 動態渲染技術。粒子在三維空間中自由移動，當距離足夠接近時會自動以動態線條連接，形成視覺上吸引人的網絡結構。

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 觀看 3D 粒子動畫 (Priority: P1)

使用者開啟頁面後，會看到一個包含數百個白色粒子在立方體邊界內隨機移動的 3D 場景。粒子會自動漂浮並反彈，形成持續流動的視覺效果。

**Why this priority**: 這是核心體驗，沒有粒子動畫就沒有視覺化展示的基礎。

**Independent Test**: 可透過開啟頁面並觀察粒子是否在 3D 空間中持續移動來驗證。

**Acceptance Scenarios**:

1. **Given** 使用者開啟頁面, **When** 頁面載入完成, **Then** 應顯示數百個白色粒子在 3D 空間中移動
2. **Given** 粒子在移動中, **When** 粒子碰到邊界, **Then** 粒子應反彈並繼續朝相反方向移動
3. **Given** 場景正在渲染, **When** 時間持續經過, **Then** 整體場景應持續平滑旋轉

---

### User Story 2 - 動態粒子連線效果 (Priority: P1)

當兩個粒子之間的距離小於設定的門檻值時，系統會自動在它們之間繪製一條線段。線段的透明度會根據距離動態調整——距離越近越不透明，距離越遠越透明。

**Why this priority**: 這是範例的核心特色，展示 drawRange 的動態渲染能力。

**Independent Test**: 觀察粒子移動時，接近的粒子之間是否會出現連線，且線條透明度隨距離變化。

**Acceptance Scenarios**:

1. **Given** 兩個粒子相距在門檻值內, **When** 渲染每一幀, **Then** 應在兩粒子間繪製連線
2. **Given** 兩個粒子正在接近, **When** 距離變短, **Then** 連線應變得更加不透明
3. **Given** 兩個粒子正在遠離, **When** 距離超過門檻值, **Then** 連線應消失

---

### User Story 3 - GUI 控制面板互動 (Priority: P2)

使用者可透過右上角的控制面板調整視覺化參數，包括顯示/隱藏粒子、顯示/隱藏連線、調整連線距離門檻、限制連接數量，以及調整粒子總數。

**Why this priority**: 提供互動控制讓使用者能探索不同的視覺效果。

**Independent Test**: 開啟控制面板，調整各項參數並觀察場景變化是否符合預期。

**Acceptance Scenarios**:

1. **Given** 控制面板已顯示, **When** 使用者切換「顯示粒子」, **Then** 粒子應顯示或隱藏
2. **Given** 控制面板已顯示, **When** 使用者切換「顯示連線」, **Then** 連線應顯示或隱藏
3. **Given** 控制面板已顯示, **When** 使用者調整「最小距離」滑桿, **Then** 連線產生的距離門檻應即時改變
4. **Given** 控制面板已顯示, **When** 使用者調整「粒子數量」滑桿, **Then** 顯示的粒子數應即時增減

---

### User Story 4 - 3D 攝影機軌道控制 (Priority: P2)

使用者可透過滑鼠或觸控手勢來旋轉、縮放和平移 3D 視角，自由探索粒子網絡的各個角度。

**Why this priority**: 軌道控制讓使用者能更深入地體驗 3D 效果。

**Independent Test**: 使用滑鼠拖曳、滾輪縮放，確認視角能平滑變化。

**Acceptance Scenarios**:

1. **Given** 場景已渲染, **When** 使用者拖曳滑鼠, **Then** 攝影機應繞著場景中心旋轉
2. **Given** 場景已渲染, **When** 使用者使用滾輪, **Then** 攝影機應縮放（有最小和最大距離限制）
3. **Given** 場景已渲染, **When** 使用者右鍵拖曳, **Then** 攝影機應平移

---

### User Story 5 - 視窗大小自適應 (Priority: P3)

當使用者調整瀏覽器視窗大小時，3D 場景應自動調整以填滿可用空間，並保持正確的長寬比。

**Why this priority**: 基本的響應式行為確保在各種螢幕尺寸下都能正常顯示。

**Independent Test**: 調整瀏覽器視窗大小，確認場景自動調整且不變形。

**Acceptance Scenarios**:

1. **Given** 場景正在顯示, **When** 使用者調整視窗大小, **Then** 渲染器應調整至新尺寸
2. **Given** 視窗大小已改變, **When** 渲染下一幀, **Then** 場景應維持正確的長寬比

---

### Edge Cases

- 當粒子數量設為 0 時，連線也應全部消失
- 當最小距離設為極大值時，所有粒子都應相互連接（效能考量下可能有上限）
- 當啟用「限制連接數」且數值很小時，每個粒子最多只產生該數量的連線
- 瀏覽器不支援 WebGL 時應顯示適當的錯誤訊息

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 在 3D 空間中渲染最多 1000 個粒子
- **FR-002**: 系統 MUST 在每一幀更新粒子位置，並在邊界反彈
- **FR-003**: 系統 MUST 使用 BufferGeometry 和 DynamicDrawUsage 以達到高效能動態更新
- **FR-004**: 系統 MUST 使用 setDrawRange 方法來控制實際渲染的粒子和線段數量
- **FR-005**: 系統 MUST 計算粒子間距離並在門檻值內動態繪製連線
- **FR-006**: 系統 MUST 根據距離計算連線的透明度（alpha 值）
- **FR-007**: 系統 MUST 顯示一個立方體輔助框來表示粒子移動的邊界
- **FR-008**: 系統 MUST 提供 GUI 控制面板包含以下控制項：
  - 顯示/隱藏粒子（showDots）
  - 顯示/隱藏連線（showLines）
  - 最小連線距離（minDistance）: 10-300
  - 限制連接數量開關（limitConnections）
  - 最大連接數量（maxConnections）: 0-30
  - 粒子數量（particleCount）: 0-1000
- **FR-009**: 系統 MUST 提供 OrbitControls 讓使用者能旋轉、縮放和平移視角
- **FR-010**: 系統 MUST 在視窗大小改變時自動調整渲染器和攝影機
- **FR-011**: 系統 MUST 使用 additive blending 和透明材質來產生發光效果
- **FR-012**: 系統 MUST 顯示效能統計資訊（FPS）

### Key Entities

- **Particle**: 代表 3D 空間中的一個點，具有位置和速度向量，以及當前連接數量
- **ParticleSystem (Points)**: 使用 BufferGeometry 儲存所有粒子位置的點雲系統
- **LineNetwork (LineSegments)**: 使用 BufferGeometry 儲存所有連線頂點的線段系統
- **EffectController**: 儲存所有可調整參數的控制物件

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 使用者開啟頁面後，在 3 秒內應能看到粒子動畫開始播放
- **SC-002**: 調整任何 GUI 控制項後，場景應在下一幀（約 16ms）內反映變化
- **SC-003**: 場景在 500 個粒子時應維持每秒 30 幀以上的效能
- **SC-004**: 視窗大小調整後，場景應在 100ms 內完成自適應
- **SC-005**: 軌道控制的操作延遲應小於 50ms，提供流暢的互動體驗

## 假設與限制

### 假設

- 使用者的瀏覽器支援 WebGL
- 使用者設備具有基本的 3D 圖形處理能力
- 使用 CDN 載入 Three.js 函式庫

### 限制

- 最大粒子數量限制為 1000 以確保效能
- 攝影機縮放距離限制在 1000-3000 單位之間
- 立方體邊界大小固定為 800 單位
