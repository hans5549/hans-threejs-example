# Feature Specification: Three.js 互動式點雲 Raycasting 範例

**Feature Branch**: `1-interactive-raycasting-points`  
**Created**: 2025-11-28  
**Status**: Draft  
**Input**: 建立 Three.js 互動式點雲 Raycasting 範例 - 展示如何使用 Raycaster 偵測滑鼠與 3D 點雲的互動

## Overview

本功能旨在建立一個互動式 3D 點雲視覺化範例，展示 Three.js 的 Raycaster 功能。使用者可以用滑鼠與三種不同類型的點雲進行互動，當滑鼠懸停在點上時會顯示視覺回饋（紅色球體標記），同時場景會自動緩慢旋轉以展示 3D 效果。

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 查看 3D 點雲視覺化 (Priority: P1)

使用者開啟網頁後，應該能夠看到三組不同顏色的 3D 點雲在場景中，並且場景會自動緩慢旋轉，讓使用者從不同角度觀察點雲的波浪形狀。

**Why this priority**: 這是核心視覺展示功能，沒有點雲顯示就沒有後續的互動功能。

**Independent Test**: 開啟網頁即可驗證，應看到紅色、綠色、青色三組點雲以波浪形狀排列，場景自動旋轉。

**Acceptance Scenarios**:

1. **Given** 使用者開啟網頁, **When** 頁面載入完成, **Then** 應顯示三組不同顏色的點雲（紅、綠、青）並排列在場景中
2. **Given** 點雲已顯示, **When** 使用者觀察畫面, **Then** 場景應持續自動繞 Y 軸緩慢旋轉
3. **Given** 點雲已顯示, **When** 使用者觀察點雲形狀, **Then** 點雲應呈現波浪狀的 3D 曲面效果

---

### User Story 2 - 滑鼠懸停互動檢測 (Priority: P1)

使用者移動滑鼠在點雲上方時，系統應能偵測到最近的點，並在該點位置顯示紅色球體作為視覺回饋，讓使用者知道哪個點被選中。

**Why this priority**: 這是本範例的核心互動功能，展示 Raycasting 技術的實際應用。

**Independent Test**: 移動滑鼠到任一點雲上方，應看到紅色球體標記出現在最接近滑鼠位置的點上。

**Acceptance Scenarios**:

1. **Given** 點雲已顯示在場景中, **When** 使用者將滑鼠移動到點雲上方, **Then** 應在最接近的點位置顯示紅色球體標記
2. **Given** 紅色球體標記已顯示, **When** 使用者繼續移動滑鼠到其他點上方, **Then** 新的紅色球體應出現在新位置，之前的球體應逐漸縮小消失
3. **Given** 使用者正在與點雲互動, **When** 滑鼠移動到點雲範圍外, **Then** 不應再產生新的標記球體

---

### User Story 3 - 效能監控顯示 (Priority: P2)

使用者應能看到即時的 FPS（每秒幀數）統計資訊，以了解動畫的執行效能。

**Why this priority**: 效能監控是開發和除錯的輔助功能，非核心互動需求。

**Independent Test**: 開啟網頁後，應在畫面角落看到 FPS 統計面板。

**Acceptance Scenarios**:

1. **Given** 網頁已載入, **When** 使用者查看畫面, **Then** 應在畫面左上角看到 FPS 統計面板
2. **Given** FPS 面板已顯示, **When** 動畫持續執行, **Then** FPS 數值應即時更新

---

### User Story 4 - 響應式視窗調整 (Priority: P3)

使用者調整瀏覽器視窗大小時，3D 場景應自動調整以適應新的視窗尺寸。

**Why this priority**: 響應式設計是良好用戶體驗的標準要求，但非核心功能。

**Independent Test**: 調整瀏覽器視窗大小，場景應正確重新渲染而不變形。

**Acceptance Scenarios**:

1. **Given** 3D 場景已顯示, **When** 使用者調整瀏覽器視窗大小, **Then** 場景應自動調整尺寸和比例以適應新視窗
2. **Given** 視窗已調整, **When** 使用者繼續與點雲互動, **Then** 滑鼠偵測功能應正常運作

---

### Edge Cases

- 當瀏覽器不支援 WebGL 時，系統應嘗試使用 Canvas 2D 降級渲染，顯示功能受限的靜態點雲圖片
- 當滑鼠快速移動時，使用時間間隔節流 (throttle) 每 20ms 更新一次偵測，避免視覺閃爍
- 當視窗小於 320x240 時，維持最小尺寸以確保點雲可見且可互動

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 在頁面載入時建立並顯示三組不同顏色的 3D 點雲（紅色、綠色、青色）
- **FR-002**: 系統 MUST 使用數學函數生成波浪狀的點雲幾何形狀
- **FR-003**: 系統 MUST 支援三種點雲類型：標準點雲、帶索引的點雲、帶分組偏移的索引點雲
- **FR-004**: 系統 MUST 持續自動旋轉攝影機或場景，以展示 3D 效果
- **FR-005**: 系統 MUST 使用 Raycaster 偵測滑鼠與點雲的交集
- **FR-006**: 系統 MUST 在偵測到交集時，於該點位置顯示紅色球體標記
- **FR-007**: 系統 MUST 支援最多 40 個球體標記的物件池機制
- **FR-008**: 系統 MUST 使球體標記隨時間逐漸縮小消失
- **FR-009**: 系統 MUST 顯示即時 FPS 效能統計資訊
- **FR-010**: 系統 MUST 在視窗大小改變時自動調整渲染尺寸和攝影機比例
- **FR-011**: 系統 MUST 使用頂點顏色（vertex colors）來渲染點雲，顏色強度應根據 Y 軸位置變化
- **FR-012**: 系統 MUST 使用單一 HTML 檔案結構，內嵌 CSS 和 JavaScript 程式碼

### Key Entities

- **PointCloud (點雲)**: 由大量 3D 點組成的幾何物件，具有位置和顏色屬性
- **Raycaster (射線投射器)**: 用於偵測滑鼠位置與 3D 物件交集的工具
- **Sphere Marker (球體標記)**: 用於標示被偵測到的點的視覺回饋物件
- **Scene (場景)**: 包含所有 3D 物件的容器
- **Camera (攝影機)**: 定義觀察視角的物件

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 使用者開啟網頁後，點雲場景應在 3 秒內完成載入並開始渲染
- **SC-002**: 動畫執行時應維持至少 30 FPS 的流暢度（在一般桌面設備上）
- **SC-003**: 滑鼠移動到點雲上方後，視覺回饋應在 100 毫秒內出現
- **SC-004**: 三組點雲應清晰可辨，各自呈現不同的顏色特徵
- **SC-005**: 視窗調整大小後，場景應在 500 毫秒內完成重新渲染
- **SC-006**: 球體標記應在約 2 秒內逐漸縮小到最小尺寸

## Clarifications

### Session 2025-11-28

- Q: 當瀏覽器不支援 WebGL 時，系統應該如何處理？ → A: 嘗試使用 Canvas 2D 降級渲染（功能受限的靜態圖片）
- Q: Three.js 函式庫應該如何載入？ → A: 使用 CDN (如 unpkg 或 jsDelivr) 直接載入
- Q: 滑鼠快速移動時應如何控制偵測更新頻率？ → A: 使用時間間隔節流 (throttle)，每 20ms 更新一次偵測
- Q: 當視窗非常小時，系統應該如何處理？ → A: 設定最小尺寸限制 (320x240)，小於此尺寸時維持最小尺寸
- Q: 此範例應該使用什麼樣的檔案結構？ → A: 單一 HTML 檔案（內嵌 CSS 和 JavaScript）

## Assumptions

- 使用者的瀏覽器支援 WebGL 2.0 或更高版本（若不支援則降級為 Canvas 2D 靜態渲染）
- 使用者的設備具備基本的 3D 渲染能力
- 網路連線正常，可從 CDN (unpkg/jsDelivr) 載入 Three.js 函式庫
- 點雲的尺寸參數為 80 x 160 = 12,800 個點（每組點雲）
- 點的大小 (pointSize) 為 0.05
- Raycaster 的偵測閾值 (threshold) 為 0.1
