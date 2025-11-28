# Feature Specification: COLLADA Kinematics 機器人手臂動畫

**Feature Branch**: `6-collada-kinematics`  
**Created**: 2025-11-28  
**Updated**: 2025-11-28  
**Status**: Draft  
**Input**: 建立 COLLADA Kinematics 機器人手臂動畫展示，使用 Three.js ColladaLoader 載入機器人模型並透過 TWEEN 動畫控制關節運動，支援切換不同的機器人樣式

## Overview

本功能將建立一個互動式 3D 機器人手臂展示，使用 Three.js 的 ColladaLoader 載入工業機器人模型，並透過 TWEEN.js 動畫引擎控制機器人的關節運動，展示機器人運動學（Kinematics）的視覺化效果。

**新增功能**：使用者可以透過選單從多種工業機器人模型中選擇，包括來自 [collada_robots](https://github.com/rdiankov/collada_robots) 儲存庫的多款機器人，如 ABB、KUKA、Universal Robots 等品牌的機器人。

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 觀看機器人自動運動 (Priority: P1)

使用者開啟網頁後，可以看到一個 3D 工業機器人手臂模型在網格地板上持續進行隨機的關節運動動畫，攝影機會自動環繞機器人旋轉，讓使用者從不同角度觀察機器人的運動。

**Why this priority**: 這是功能的核心價值，展示機器人運動學動畫是本功能的主要目的

**Independent Test**: 可以直接開啟網頁，觀察機器人是否正確載入並開始自動運動動畫

**Acceptance Scenarios**:

1. **Given** 使用者開啟網頁, **When** COLLADA 模型載入完成, **Then** 機器人模型正確顯示在畫面中央的網格地板上
2. **Given** 機器人模型已載入, **When** 動畫開始執行, **Then** 機器人的各個關節開始平滑地移動到隨機目標位置
3. **Given** 動畫正在執行, **When** 一個運動週期完成, **Then** 系統自動設定新的隨機目標位置並開始下一個運動週期

---

### User Story 2 - 環繞視角觀察 (Priority: P1)

使用者可以從持續旋轉的攝影機視角觀察機器人，攝影機會自動環繞機器人中心點旋轉，提供 360 度的觀察體驗。

**Why this priority**: 自動環繞攝影機是展示 3D 模型的重要功能，讓使用者無需手動操作即可觀察模型全貌

**Independent Test**: 開啟網頁後觀察攝影機是否自動環繞機器人旋轉

**Acceptance Scenarios**:

1. **Given** 場景已初始化, **When** 渲染迴圈執行, **Then** 攝影機沿著圓形軌道環繞機器人旋轉
2. **Given** 攝影機正在旋轉, **When** 任何時間點, **Then** 攝影機始終看向機器人中心位置

---

### User Story 3 - 效能監控 (Priority: P2)

使用者可以看到即時的效能統計資訊（FPS），了解動畫的執行效能。

**Why this priority**: 效能監控對於開發者和進階使用者有價值，但非核心功能

**Independent Test**: 檢查網頁左上角是否顯示 FPS 計數器

**Acceptance Scenarios**:

1. **Given** 網頁已載入, **When** 動畫執行中, **Then** 左上角顯示即時的 FPS 統計數據

---

### User Story 4 - 響應式視窗調整 (Priority: P2)

當使用者調整瀏覽器視窗大小時，3D 場景自動調整以適應新的視窗尺寸，保持正確的長寬比例。

**Why this priority**: 響應式設計確保在不同裝置和視窗大小下的良好體驗

**Independent Test**: 調整瀏覽器視窗大小，觀察 3D 場景是否正確調整

**Acceptance Scenarios**:

1. **Given** 網頁正在顯示, **When** 使用者調整視窗大小, **Then** 渲染畫布自動調整為新的視窗尺寸
2. **Given** 視窗大小已改變, **When** 場景重新渲染, **Then** 畫面保持正確的長寬比例，無拉伸或變形

---

### User Story 5 - 切換機器人樣式 (Priority: P1)

使用者可以透過下拉選單從多種工業機器人模型中選擇想要觀看的機器人，選擇後系統會載入對應的機器人模型並繼續運動學動畫。

**Why this priority**: 這是本次新增的核心功能，讓使用者可以探索不同類型的工業機器人，大幅提升展示的教育價值和互動性

**Independent Test**: 點擊選單選擇不同的機器人，觀察場景是否正確切換模型

**Acceptance Scenarios**:

1. **Given** 網頁已載入, **When** 使用者查看機器人選單, **Then** 選單顯示至少 5 種不同的機器人選項
2. **Given** 使用者選擇新的機器人, **When** 選擇確認後, **Then** 系統移除當前機器人並載入新選擇的機器人模型
3. **Given** 新機器人載入完成, **When** 模型顯示在場景中, **Then** 運動學動畫自動重新啟動，使用新機器人的關節配置
4. **Given** 載入過程進行中, **When** 使用者等待, **Then** 顯示載入進度或指示

---

### User Story 6 - 顯示機器人資訊 (Priority: P3)

使用者可以看到當前選擇的機器人的基本資訊，如名稱、製造商、關節數量等。

**Why this priority**: 提供額外的教育資訊，但非核心展示功能

**Independent Test**: 選擇機器人後檢查是否顯示相關資訊

**Acceptance Scenarios**:

1. **Given** 機器人已載入, **When** 使用者查看資訊區域, **Then** 顯示當前機器人的名稱和製造商

---

### Edge Cases

- 當 COLLADA 模型檔案載入失敗時，系統應優雅地處理錯誤並提示使用者
- 當瀏覽器不支援 WebGL 時，應顯示適當的提示訊息
- 當視窗極小時，場景仍應正確渲染
- 當使用者快速連續切換機器人時，系統應取消前一個載入請求，避免載入衝突
- 當選擇的機器人模型不支援運動學時，系統應顯示靜態模型並通知使用者

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST 使用 ColladaLoader 載入 COLLADA 格式的機器人 DAE 模型檔案
- **FR-002**: System MUST 解析並套用 COLLADA 檔案中的運動學（Kinematics）資訊
- **FR-003**: System MUST 使用 TWEEN.js 建立平滑的關節運動動畫
- **FR-004**: System MUST 在每個動畫週期完成後，自動產生新的隨機關節目標位置
- **FR-005**: System MUST 確保關節運動在定義的最小值和最大值範圍內
- **FR-006**: System MUST 渲染一個 20x20 單位的網格地板作為參考平面
- **FR-007**: System MUST 使用半球光源提供均勻的場景照明
- **FR-008**: System MUST 實作自動環繞攝影機，以圓形軌道環繞場景中心旋轉
- **FR-009**: System MUST 使用 FlatShading 材質渲染機器人模型
- **FR-010**: System MUST 將機器人模型縮放至適當大小以便觀察
- **FR-011**: System MUST 顯示即時 FPS 統計資訊
- **FR-012**: System MUST 在視窗大小改變時自動調整渲染器和攝影機設定
- **FR-013**: System MUST 提供下拉選單讓使用者選擇不同的機器人模型
- **FR-014**: System MUST 支援至少以下機器人模型選項：
  - ABB IRB 52（預設）
  - KUKA KR5 R650
  - KUKA KR5 R850
  - KUKA YouBot
  - Universal Robots UR6
- **FR-015**: System MUST 在切換機器人時平滑地移除舊模型並載入新模型
- **FR-016**: System MUST 在載入新模型時顯示載入狀態指示
- **FR-017**: System MUST 在新模型載入完成後自動重新配置運動學動畫
- **FR-018**: System MUST 顯示當前選擇的機器人名稱

### Key Entities

- **Scene**: Three.js 場景容器，包含所有 3D 物件
- **Robot Model (DAE)**: COLLADA 格式的工業機器人 3D 模型
- **Robot Catalog**: 可選擇的機器人模型清單，包含名稱、檔案路徑、製造商資訊
- **Kinematics**: 從 COLLADA 檔案解析的運動學資訊，包含關節定義和限制
- **Joint**: 機器人的可動關節，具有最小值、最大值和當前位置屬性
- **Camera**: 透視投影攝影機，自動環繞場景旋轉
- **GridHelper**: 網格地板輔助物件，提供空間參考
- **HemisphereLight**: 半球光源，提供場景照明
- **Robot Selector UI**: 機器人選擇介面元件（下拉選單）

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 機器人模型在網頁載入後 5 秒內完成載入並開始動畫
- **SC-002**: 動畫在主流瀏覽器（Chrome, Firefox, Safari, Edge）上以 30 FPS 以上的速度流暢執行
- **SC-003**: 使用者可以在 10 秒內從多個角度觀察到機器人的完整外觀（透過自動環繞攝影機）
- **SC-004**: 每個關節運動週期持續 1-5 秒，提供自然的運動節奏
- **SC-005**: 視窗調整大小後，畫面在 0.5 秒內完成重新渲染
- **SC-006**: 使用者切換機器人後，新模型在 5 秒內完成載入並開始動畫
- **SC-007**: 機器人選單提供至少 5 種不同的機器人選項
- **SC-008**: 90% 的使用者能在首次使用時成功切換機器人模型（無需說明）

### Technical Assumptions

- 使用者的瀏覽器支援 WebGL
- 使用者的網路連線可以載入 COLLADA 模型檔案（檔案大小不等）
- 使用者的裝置具備基本的 3D 圖形處理能力
- 部分機器人模型可能不包含完整的運動學資訊，此時僅顯示靜態模型

## Available Robot Models

以下是來自 [collada_robots](https://github.com/rdiankov/collada_robots) 儲存庫的可用機器人模型：

| 機器人名稱 | 製造商 | 檔案名稱 | 說明 |
|-----------|--------|----------|------|
| ABB IRB 52 | ABB | abb_irb52_7_120.dae | 小型工業機器人手臂（預設） |
| KUKA KR5 R650 | KUKA | kuka-kr5-r650.zae | 中型工業機器人 |
| KUKA KR5 R850 | KUKA | kuka-kr5-r850.zae | 中型工業機器人（加長版） |
| KUKA YouBot | KUKA | kuka-youbot.zae | 移動式研究機器人 |
| Universal Robots UR6 | Universal Robots | universalrobots-ur6-85-5-a.zae | 協作機器人 |
| Barrett WAM | Barrett | barrett-wam.zae | 研究用機器人手臂 |
| Shadow Hand | Shadow Robot | shadow-hand.zae | 仿人機器人手 |
| PR2 | Willow Garage | willowgarage-pr2.zae | 個人服務機器人 |
