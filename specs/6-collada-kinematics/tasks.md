# Tasks: COLLADA Kinematics 機器人手臂動畫

**Input**: Design documents from `/specs/6-collada-kinematics/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/component-interface.md ✅

**Tests**: 本功能未要求自動化測試，採用手動驗證方式。

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Setup (專案初始化) ✅ COMPLETED

**Purpose**: 建立專案基礎結構和下載模型檔案

- [x] T001 建立專案目錄結構 `examples/collada-kinematics/` 和 `examples/collada-kinematics/models/`
- [x] T002 [P] 下載 ABB IRB 52 模型至 `examples/collada-kinematics/models/abb_irb52_7_120.dae`
- [x] T003 [P] 下載並解壓縮 KUKA KR5-R650 模型至 `examples/collada-kinematics/models/kuka-kr5-r650.dae`
- [x] T004 [P] 下載並解壓縮 Universal Robots UR6 模型至 `examples/collada-kinematics/models/universalrobots-ur6-85-5-a.dae`
- [x] T005 [P] 建立 WebGL fallback 圖片 `examples/collada-kinematics/fallback.svg`

**Checkpoint**: ✅ 專案目錄結構完成，所有模型檔案就緒

---

## Phase 2: Foundational (基礎框架) ✅ COMPLETED

**Purpose**: 建立 HTML 骨架、CSS 樣式和核心 Three.js 初始化

**⚠️ CRITICAL**: 所有 User Story 都依賴此階段完成

- [x] T006 建立 HTML 基本結構含 importmap 和 meta 標籤於 `examples/collada-kinematics/index.html`
- [x] T007 實作內嵌 CSS 樣式（body reset、loading overlay、selector、info panel）於 `examples/collada-kinematics/index.html`
- [x] T008 實作 WebGL 支援檢測和降級處理於 `examples/collada-kinematics/index.html`
- [x] T009 實作 Scene 模組：建立場景、渲染器、光源、地板網格於 `examples/collada-kinematics/index.html`
- [x] T010 實作 Camera 模組：建立透視攝影機和 onResize 處理於 `examples/collada-kinematics/index.html`
- [x] T011 實作主要渲染迴圈 `animate()` 函式於 `examples/collada-kinematics/index.html`

**Checkpoint**: ✅ 基礎 Three.js 場景可渲染空場景，視窗縮放正常運作

---

## Phase 3: User Story 1 - 觀看機器人自動運動 (Priority: P1) 🎯 MVP ✅ COMPLETED

**Goal**: 載入機器人模型並展示自動關節運動動畫

**Independent Test**: 開啟網頁，確認機器人模型正確載入並開始隨機關節運動

### Implementation for User Story 1

- [x] T012 [US1] 定義 ROBOT_CATALOG 靜態資料（3 種機器人配置）於 `examples/collada-kinematics/index.html`
- [x] T013 [US1] 實作 Robot.load() 使用 ColladaLoader 載入模型於 `examples/collada-kinematics/index.html`
- [x] T014 [US1] 實作模型後處理：FlatShading 材質、縮放定位於 `examples/collada-kinematics/index.html`
- [x] T015 [US1] 實作運動學資料解析：提取 joints 和 limits 於 `examples/collada-kinematics/index.html`
- [x] T016 [US1] 實作 Animation.setupTween() 產生隨機目標關節值於 `examples/collada-kinematics/index.html`
- [x] T017 [US1] 實作 TWEEN 動畫循環：完成後自動重新觸發於 `examples/collada-kinematics/index.html`
- [x] T018 [US1] 整合 Animation.update() 至渲染迴圈於 `examples/collada-kinematics/index.html`
- [x] T019 [US1] 處理缺少運動學資料的情況（顯示靜態模型）於 `examples/collada-kinematics/index.html`

**Checkpoint**: ✅ ABB 機器人載入並執行隨機關節動畫，動畫 1-5 秒週期循環

---

## Phase 4: User Story 2 - 環繞視角觀察 (Priority: P1) ✅ COMPLETED

**Goal**: 攝影機自動環繞機器人旋轉，提供 360 度觀察

**Independent Test**: 觀察攝影機是否持續環繞機器人旋轉，始終看向中心

### Implementation for User Story 2

- [x] T020 [US2] 實作 Camera.updateOrbit() 環繞動畫邏輯於 `examples/collada-kinematics/index.html`
- [x] T021 [US2] 整合環繞動畫至渲染迴圈於 `examples/collada-kinematics/index.html`
- [x] T022 [US2] 設定攝影機初始位置和目標點於 `examples/collada-kinematics/index.html`

**Checkpoint**: ✅ 攝影機自動環繞機器人，可從各角度觀察模型

---

## Phase 5: User Story 5 - 切換機器人樣式 (Priority: P1) ✅ COMPLETED

**Goal**: 使用者可透過下拉選單切換不同機器人模型

**Independent Test**: 點擊選單選擇不同機器人，確認模型正確切換並重啟動畫

### Implementation for User Story 5

- [x] T023 [US5] 建立機器人選擇器 HTML 元素（右上角 select）於 `examples/collada-kinematics/index.html`
- [x] T024 [US5] 實作 UI.init() 動態填充選擇器選項於 `examples/collada-kinematics/index.html`
- [x] T025 [US5] 實作 Robot.unload() 移除當前模型於 `examples/collada-kinematics/index.html`
- [x] T026 [US5] 實作 Robot.switchTo() 整合載入/卸載流程於 `examples/collada-kinematics/index.html`
- [x] T027 [US5] 建立載入進度指示器 HTML 元素（中央 overlay）於 `examples/collada-kinematics/index.html`
- [x] T028 [US5] 實作 UI.showLoading() 和 UI.hideLoading() 於 `examples/collada-kinematics/index.html`
- [x] T029 [US5] 實作載入進度回呼更新百分比顯示於 `examples/collada-kinematics/index.html`
- [x] T030 [US5] 綁定選擇器 change 事件觸發模型切換於 `examples/collada-kinematics/index.html`
- [x] T031 [US5] 處理快速連續切換的情況（取消前一個載入）於 `examples/collada-kinematics/index.html`

**Checkpoint**: ✅ 可切換 3 種機器人，載入時顯示進度，切換後動畫自動重啟

---

## Phase 6: User Story 3 - 效能監控 (Priority: P2) ✅ COMPLETED

**Goal**: 顯示即時 FPS 統計資訊

**Independent Test**: 檢查左上角是否顯示 FPS 計數器

### Implementation for User Story 3

- [x] T032 [US3] 引入 Stats 模組並初始化於 `examples/collada-kinematics/index.html`
- [x] T033 [US3] 將 Stats DOM 元素加入頁面於 `examples/collada-kinematics/index.html`
- [x] T034 [US3] 整合 stats.update() 至渲染迴圈於 `examples/collada-kinematics/index.html`

**Checkpoint**: ✅ 左上角顯示即時 FPS 統計

---

## Phase 7: User Story 4 - 響應式視窗調整 (Priority: P2) ✅ COMPLETED

**Goal**: 視窗大小改變時自動調整場景

**Independent Test**: 調整瀏覽器視窗大小，確認場景正確調整無變形

### Implementation for User Story 4

- [x] T035 [US4] 實作 window resize 事件監聽器於 `examples/collada-kinematics/index.html`
- [x] T036 [US4] 更新攝影機 aspect ratio 和 renderer size 於 `examples/collada-kinematics/index.html`

**Checkpoint**: ✅ 視窗縮放時場景正確調整，保持正確長寬比

---

## Phase 8: User Story 6 - 顯示機器人資訊 (Priority: P3) ✅ COMPLETED

**Goal**: 顯示當前機器人的基本資訊

**Independent Test**: 選擇機器人後檢查左下角是否顯示名稱和製造商

### Implementation for User Story 6

- [x] T037 [US6] 建立機器人資訊面板 HTML 元素（左下角）於 `examples/collada-kinematics/index.html`
- [x] T038 [US6] 實作 UI.updateRobotInfo() 更新顯示內容於 `examples/collada-kinematics/index.html`
- [x] T039 [US6] 在模型載入完成後呼叫資訊更新於 `examples/collada-kinematics/index.html`

**Checkpoint**: ✅ 左下角顯示當前機器人名稱和製造商

---

## Phase 9: Polish & 整合測試 ✅ COMPLETED

**Purpose**: 最終驗證和品質確認

- [x] T040 錯誤處理完善：模型載入失敗時顯示友善訊息於 `examples/collada-kinematics/index.html`
- [x] T041 [P] 程式碼清理和註解完善於 `examples/collada-kinematics/index.html`
- [x] T042 [P] 執行 quickstart.md 驗證所有功能
- [x] T043 驗證所有 Success Criteria (SC-001 至 SC-008)

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1 (Setup)
    ↓
Phase 2 (Foundational)
    ↓
┌───────────────────────────────────────────────────┐
│  Phase 3 (US1) → Phase 4 (US2) → Phase 5 (US5)   │ ← P1 User Stories
│         ↓              ↓              ↓           │
│  Phase 6 (US3)   Phase 7 (US4)   Phase 8 (US6)   │ ← P2/P3 User Stories
└───────────────────────────────────────────────────┘
    ↓
Phase 9 (Polish)
```

### User Story Dependencies

| User Story | Phase | 依賴 | 說明 |
|------------|-------|------|------|
| US1 (機器人運動) | 3 | Phase 2 | MVP 核心功能 |
| US2 (環繞視角) | 4 | Phase 2 | 可與 US1 平行 |
| US5 (切換機器人) | 5 | US1 | 需要載入邏輯先完成 |
| US3 (FPS 監控) | 6 | Phase 2 | 獨立功能 |
| US4 (響應式) | 7 | Phase 2 | 獨立功能 |
| US6 (機器人資訊) | 8 | US5 | 需要切換功能先完成 |

### Parallel Opportunities

**Phase 1 平行任務**:
```bash
# 可同時執行：
T002 下載 ABB 模型
T003 下載 KUKA 模型
T004 下載 UR6 模型
T005 建立 fallback.svg
```

**Phase 2 後平行機會**:
```bash
# Phase 2 完成後，以下可同時進行：
- US1 (Phase 3): 機器人載入和動畫
- US2 (Phase 4): 環繞攝影機
- US3 (Phase 6): FPS 監控
- US4 (Phase 7): 響應式視窗
```

---

## Implementation Strategy

### MVP First (Phase 1-3)

1. ✅ Complete Phase 1: Setup（模型下載）
2. ✅ Complete Phase 2: Foundational（Three.js 基礎）
3. ✅ Complete Phase 3: User Story 1（機器人運動）
4. **STOP and VALIDATE**: 測試 ABB 機器人是否正確載入和運動
5. 可部署 MVP 版本（僅支援 ABB 機器人）

### Incremental Delivery

1. MVP: Phase 1-3 → ABB 機器人自動運動
2. +US2: Phase 4 → 環繞攝影機
3. +US5: Phase 5 → 3 種機器人切換
4. +US3/US4: Phase 6-7 → FPS 和響應式
5. +US6: Phase 8 → 機器人資訊顯示
6. Polish: Phase 9 → 最終驗證

### 單一 HTML 檔案開發流程

由於本功能採用單一 `index.html` 檔案，建議：

1. **逐步累加**: 從 HTML 骨架開始，逐步加入各模組程式碼
2. **即時測試**: 每完成一個任務即在瀏覽器測試
3. **模組化組織**: 使用註解區塊分隔不同模組
4. **版本控制**: 每個 Phase 完成後提交

---

## Success Criteria 驗證清單

| ID | 標準 | 驗證方式 |
|----|------|----------|
| SC-001 | 模型 5 秒內載入完成 | 使用 DevTools Network 計時 |
| SC-002 | 30+ FPS 流暢執行 | 檢查 Stats 面板 |
| SC-003 | 10 秒內觀察完整外觀 | 目測環繞攝影機一圈 |
| SC-004 | 動畫 1-5 秒週期 | 目測動畫節奏 |
| SC-005 | 視窗調整 0.5 秒內完成 | 目測響應速度 |
| SC-006 | 切換機器人 5 秒內完成 | 使用計時器測量 |
| SC-007 | 提供 3 種機器人選項 | 檢查下拉選單 |
| SC-008 | 90% 使用者可成功切換 | UI 直覺性評估 |

---

## Notes

- 所有程式碼位於單一 `examples/collada-kinematics/index.html` 檔案
- 模型檔案需預先下載至 `models/` 目錄
- 使用 CDN 載入 Three.js 無需 npm 安裝
- 測試需使用本地 HTTP 伺服器（避免 CORS）
