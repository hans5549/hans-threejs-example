# Feature Specification: WebGL Geometry Terrain

**Feature Branch**: `016-geometry-terrain`  
**Created**: 2025-12-04  
**Status**: Planned  
**Input**: User description: "WebGL geometry terrain with procedural height map, fog effect, first-person controls, and dynamic texture generation based on Three.js official example"

**Reference**: [Three.js webgl_geometry_terrain](https://threejs.org/examples/#webgl_geometry_terrain)

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 探索程序化生成的 3D 地形 (Priority: P1)

作為使用者，我希望能夠在瀏覽器中看到一個程序化生成的 3D 地形場景，讓我能夠體驗沉浸式的地形環境。

**Why this priority**: 這是功能的核心價值 - 沒有可視化的地形，其他所有功能都無法展示。地形生成是整個範例的基礎。

**Independent Test**: 可以透過開啟頁面並觀察是否顯示起伏的 3D 地形來獨立測試。即使沒有控制功能，靜態地形也能展示程序化生成的效果。

**Acceptance Scenarios**:

1. **Given** 使用者開啟範例頁面，**When** 頁面載入完成，**Then** 應顯示一個具有高低起伏的 3D 地形網格
2. **Given** 地形已渲染，**When** 使用者觀察場景，**Then** 地形應具有基於高度的明暗著色紋理
3. **Given** 地形已渲染，**When** 使用者觀察遠處，**Then** 應看到霧效果使遠處地形逐漸融入背景

---

### User Story 2 - 使用第一人稱視角探索地形 (Priority: P2)

作為使用者，我希望能夠使用第一人稱控制器在地形中自由移動和觀看，讓我能夠從不同角度探索地形的細節。

**Why this priority**: 互動性是 3D 展示的重要組成部分，但需要先有地形才能探索。

**Independent Test**: 可以透過使用滑鼠和鍵盤操作來測試，驗證視角是否能夠流暢移動和轉向。

**Acceptance Scenarios**:

1. **Given** 地形場景已載入，**When** 使用者按下左鍵或前進鍵，**Then** 視角應向前移動
2. **Given** 地形場景已載入，**When** 使用者按下右鍵或後退鍵，**Then** 視角應向後移動
3. **Given** 地形場景已載入，**When** 使用者移動滑鼠，**Then** 視角應跟隨滑鼠方向旋轉
4. **Given** 使用者正在移動視角，**When** 移動速度保持穩定，**Then** 視角移動應流暢無卡頓

---

### User Story 3 - 體驗視覺效果增強 (Priority: P3)

作為使用者，我希望場景具有視覺效果增強，包括指數霧效果和動態紋理，讓整體視覺體驗更加真實和沉浸。

**Why this priority**: 視覺效果增強改善使用者體驗，但不是核心功能。

**Independent Test**: 可以透過觀察遠近物體的霧效果差異和地形紋理的自然變化來測試。

**Acceptance Scenarios**:

1. **Given** 場景已渲染，**When** 使用者觀察不同距離的地形，**Then** 越遠的地形應越模糊，融入霧色背景
2. **Given** 地形紋理已生成，**When** 使用者觀察地形表面，**Then** 應看到基於光照方向計算的明暗變化
3. **Given** 地形紋理已生成，**When** 使用者仔細觀察，**Then** 應看到細微的噪點增加紋理的自然感

---

### User Story 4 - 響應式視窗調整 (Priority: P4)

作為使用者，我希望當調整瀏覽器視窗大小時，場景能夠正確適應新的尺寸。

**Why this priority**: 響應式設計改善使用體驗，但不影響核心功能。

**Independent Test**: 可以透過調整瀏覽器視窗大小來測試渲染器是否正確更新。

**Acceptance Scenarios**:

1. **Given** 場景已渲染，**When** 使用者調整瀏覽器視窗大小，**Then** 場景應自動適應新的視窗尺寸
2. **Given** 視窗大小已改變，**When** 渲染器更新，**Then** 場景比例應保持正確無變形

---

### Edge Cases

- 當瀏覽器不支援 WebGL 時，系統應降級為 2D Canvas 顯示靜態地形圖
- 當視窗尺寸低於最小閾值 (200x200 像素) 時，停止渲染更新以避免異常行為
- 當使用者快速連續點擊左右鍵時，移動基於時間差計算確保行為一致
- 當頁面在背景標籤時，依賴瀏覽器 requestAnimationFrame 自動暫停動畫迴圈

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統必須使用 ImprovedNoise 演算法生成程序化高度圖資料
- **FR-002**: 系統必須建立一個 PlaneGeometry 並根據高度圖資料設定頂點高度
- **FR-003**: 系統必須動態生成基於高度和光照的紋理
- **FR-004**: 系統必須實作指數霧效果 (FogExp2) 以增加場景深度感
- **FR-005**: 系統必須實作 FirstPersonControls 讓使用者能夠自由探索場景
- **FR-006**: 系統必須支援左鍵前進、右鍵後退的操作方式
- **FR-007**: 系統必須在視窗大小改變時更新相機比例和渲染器尺寸
- **FR-008**: 系統必須顯示 Stats 效能監控面板
- **FR-009**: 系統必須使用 setAnimationLoop 進行渲染迴圈管理

### Key Entities

- **Height Map (高度圖)**: 一個 256x256 的數值陣列，代表地形每個點的高度值，由 ImprovedNoise 演算法生成
- **Terrain Mesh (地形網格)**: 一個 PlaneGeometry 網格，頂點 Y 座標根據高度圖調整，代表實際的 3D 地形
- **Terrain Texture (地形紋理)**: 一個 Canvas 生成的紋理，根據高度和光照方向計算明暗
- **First Person Controls (第一人稱控制器)**: 處理使用者輸入並控制相機移動和旋轉的控制系統
- **Scene Environment (場景環境)**: 包含背景色、霧效果的場景設定

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 使用者開啟頁面後，地形應在 3 秒內完成載入並顯示
- **SC-002**: 第一人稱控制器應支援每秒 150 單位的移動速度
- **SC-003**: 場景應在 60 FPS 下流暢運行（在一般桌面電腦上）
- **SC-004**: 視窗調整大小後，場景應在 100 毫秒內完成適應
- **SC-005**: 高度圖生成應產生可辨識的山脈和谷地地形特徵
- **SC-006**: 霧效果應使超過一定距離的物體逐漸融入背景，增加深度感

## Assumptions

- 使用者的瀏覽器支援 WebGL
- 使用者有滑鼠或觸控板進行互動
- 地形世界尺寸為 256x256 網格，物理尺寸為 7500x7500 單位

## Clarifications

### Session 2025-12-04

- Q: 當瀏覽器不支援 WebGL 時應如何處理？ → A: 降級為 2D Canvas 顯示靜態地形圖
- Q: 當頁面在背景標籤時動畫迴圈如何處理？ → A: 依賴瀏覽器 requestAnimationFrame 自動暫停
- Q: 當視窗尺寸極小時控制器如何處理？ → A: 設定最小閾值 200x200，低於此值停止渲染更新
- Q: 當使用者快速連續點擊左右鍵時移動行為如何？ → A: 基於時間差計算，確保一致性與平滑過渡
- 高度值會乘以 10 作為實際 Y 座標
- 背景色和霧色使用暖色調 (#efd1b5)
