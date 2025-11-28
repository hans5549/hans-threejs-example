````markdown
# Tasks: WebGL Spotlight 互動展示

**Feature**: `5-webgl-spotlight`  
**Input**: Design documents from `/specs/5-webgl-spotlight/`  
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/component-interface.md ✅

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2)
- Include exact file paths in descriptions

## Path Conventions

```text
examples/
└── webgl-spotlight/
    ├── index.html       # 主要 HTML 檔案 (內嵌 CSS + JS)
    └── fallback.png     # WebGL 不支援時的降級靜態圖片
```

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: 專案初始化和基礎結構

- [x] T001 建立專案目錄結構 `examples/webgl-spotlight/`
- [x] T002 [P] 建立 WebGL 降級靜態圖片 `examples/webgl-spotlight/fallback.svg`
- [x] T003 建立 HTML 基礎骨架與 Import Map 配置 `examples/webgl-spotlight/index.html`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 必須完成的核心基礎設施，所有 User Story 都依賴此階段

**⚠️ CRITICAL**: 此階段完成前，任何 User Story 都無法開始

- [x] T004 實作 WebGL 支援偵測函數 `checkWebGLSupport()` in `index.html`
- [x] T005 實作 WebGL 不支援時的降級處理邏輯 in `index.html`
- [x] T006 [P] 實作 `createRenderer()` 函數 - 建立 WebGLRenderer 並配置抗鋸齒、色調映射、陰影 in `index.html`
- [x] T007 [P] 實作 `createCamera()` 函數 - 建立 PerspectiveCamera (fov:40, position:7,4,1) in `index.html`
- [x] T008 [P] 實作 `loadTextures()` 函數 - 從 CDN 載入 3 張紋理 (disturb.jpg, colors.png, uv_grid_opengl.jpg) in `index.html`
- [x] T009 實作 `onWindowResize()` 函數 - 處理視窗大小變更事件 in `index.html`
- [x] T010 實作 `animate()` 動畫主迴圈骨架 (不含光源移動邏輯) in `index.html`

**Checkpoint**: 基礎設施就緒 - 可以開始實作 User Story

---

## Phase 3: User Story 1 - 基礎場景與聚光燈展示 (Priority: P1) 🎯 MVP

**Goal**: 使用者開啟網頁後，能看到包含聚光燈照明的 3D 場景，聚光燈自動環繞移動展示動態光影效果

**Independent Test**: 開啟頁面後，無需任何操作即可看到聚光燈照射的 3D 模型和動態移動的光源產生的陰影變化

### Implementation for User Story 1

- [x] T011 [P] [US1] 建立 Scene 物件並設定背景 in `index.html`
- [x] T012 [P] [US1] 實作 `createGroundPlane()` 函數 - 建立地面平面 (10x10, y:-1, 接收陰影) in `index.html`
- [x] T013 [P] [US1] 實作 `createFallbackModel()` 函數 - 建立後備球體幾何體 in `index.html`
- [x] T014 [US1] 實作 `loadLucyModel()` 函數 - 使用 PLYLoader 從 CDN 載入 Lucy100k.ply 模型 in `index.html`
- [x] T015 [US1] 為 `loadLucyModel()` 加入錯誤處理 - 載入失敗時自動使用後備球體 in `index.html`
- [x] T016 [US1] 實作 `createSpotLight()` 函數 - 建立聚光燈 (color:0xffffff, intensity:100, castShadow:true) in `index.html`
- [x] T017 [US1] 配置聚光燈陰影參數 (mapSize:1024, bias:-0.003, PCFSoftShadowMap) in `index.html`
- [x] T018 [US1] 實作聚光燈紋理投射 - 設定 spotLight.map 為預設紋理 (disturb.jpg) in `index.html`
- [x] T019 [US1] 建立 HemisphereLight 環境光 (sky:0xffffff, ground:0x8d8d8d, intensity:0.25) in `index.html`
- [x] T020 [US1] 實作 `animate()` 中的聚光燈環繞移動邏輯 (cos/sin, 半徑 2.5, 高度 5) in `index.html`
- [x] T021 [US1] 實作 `init()` 主初始化函數 - 整合所有元件建立流程 in `index.html`

**Checkpoint**: User Story 1 完成 - 開啟頁面即可看到動態光影效果的完整 3D 場景

---

## Phase 4: User Story 2 - 攝影機互動控制 (Priority: P1)

**Goal**: 使用者能夠使用滑鼠操控攝影機視角，從不同角度觀察光影效果

**Independent Test**: 使用滑鼠拖曳、滾輪縮放，確認攝影機能自由旋轉和調整距離

### Implementation for User Story 2

- [x] T022 [US2] 實作 `createControls()` 函數 - 建立 OrbitControls 並綁定至 camera 和 renderer.domElement in `index.html`
- [x] T023 [US2] 配置 OrbitControls 距離限制 (minDistance:2, maxDistance:10) in `index.html`
- [x] T024 [US2] 配置 OrbitControls 極角限制 (maxPolarAngle: Math.PI/2) - 防止攝影機低於地平面 in `index.html`
- [x] T025 [US2] 設定 OrbitControls 目標點 (target: 0,1,0) in `index.html`
- [x] T026 [US2] 在 `init()` 中整合 OrbitControls 建立流程 in `index.html`

**Checkpoint**: User Story 2 完成 - 使用者可自由旋轉和縮放攝影機視角

---

## Phase 5: User Story 3 - GUI 控制面板調整光源屬性 (Priority: P2)

**Goal**: 使用者能透過 GUI 即時調整聚光燈屬性（顏色、強度、角度、衰減等）

**Independent Test**: 調整 GUI 中的任一參數，確認光照效果即時變化

### Implementation for User Story 3

- [x] T027 [US3] 定義 GUI 參數物件 (params) - 包含所有可調整屬性的初始值 in `index.html`
- [x] T028 [US3] 實作 `createGUI()` 函數骨架 - 建立 lil-gui 實例 in `index.html`
- [x] T029 [P] [US3] 加入 GUI 顏色控制項 (color) - 使用 addColor() 並綁定 onChange 更新 spotLight.color in `index.html`
- [x] T030 [P] [US3] 加入 GUI 強度滑桿 (intensity: 0-500) - 綁定 onChange 更新 spotLight.intensity in `index.html`
- [x] T031 [P] [US3] 加入 GUI 距離滑桿 (distance: 0-20) - 綁定 onChange 更新 spotLight.distance in `index.html`
- [x] T032 [P] [US3] 加入 GUI 角度滑桿 (angle: 0-60°) - 綁定 onChange 更新 spotLight.angle in `index.html`
- [x] T033 [P] [US3] 加入 GUI 邊緣模糊滑桿 (penumbra: 0-1) - 綁定 onChange 更新 spotLight.penumbra in `index.html`
- [x] T034 [P] [US3] 加入 GUI 衰減滑桿 (decay: 1-2) - 綁定 onChange 更新 spotLight.decay in `index.html`
- [x] T035 [US3] 在 `init()` 中整合 GUI 建立流程 in `index.html`

**Checkpoint**: User Story 3 完成 - 使用者可即時調整 6 項光源屬性

---

## Phase 6: User Story 4 - 紋理投射功能 (Priority: P2)

**Goal**: 使用者能透過 GUI 選擇不同紋理貼圖應用於聚光燈，產生投影機效果

**Independent Test**: 在 GUI 中切換不同紋理，確認投射效果即時改變

### Implementation for User Story 4

- [x] T036 [US4] 加入 GUI 紋理下拉選單 (map) - 使用 add() 配合 textures 物件 in `index.html`
- [x] T037 [US4] 實作紋理切換 onChange 邏輯 - 更新 spotLight.map 屬性 in `index.html`
- [x] T038 [US4] 處理「無紋理」選項 - 當選擇 none 時設定 spotLight.map = null in `index.html`

**Checkpoint**: User Story 4 完成 - 使用者可切換 4 種紋理投射選項

---

## Phase 7: User Story 5 - 陰影參數調整 (Priority: P3)

**Goal**: 使用者能透過 GUI 調整陰影相關參數，觀察陰影品質變化

**Independent Test**: 調整陰影焦點和強度參數，確認陰影效果變化

### Implementation for User Story 5

- [x] T039 [P] [US5] 加入 GUI 陰影聚焦滑桿 (focus: 0-1) - 綁定 onChange 更新 spotLight.shadow.focus in `index.html`
- [x] T040 [P] [US5] 加入 GUI 陰影強度滑桿 (shadowIntensity: 0-1) - 綁定 onChange 更新 spotLight.shadow.intensity in `index.html`

**Checkpoint**: User Story 5 完成 - 使用者可調整 2 項陰影參數

---

## Phase 8: User Story 6 - 輔助視覺化工具 (Priority: P3)

**Goal**: 使用者能開啟/關閉光源輔助線和陰影攝影機輔助線

**Independent Test**: 切換 helpers 開關，確認輔助線的顯示與隱藏

### Implementation for User Story 6

- [x] T041 [US6] 建立 SpotLightHelper 並加入場景 (初始隱藏) in `index.html`
- [x] T042 [US6] 建立 CameraHelper (shadowCameraHelper) 並加入場景 (初始隱藏) in `index.html`
- [x] T043 [US6] 加入 GUI helpers 開關控制項 - 綁定 onChange 切換兩個 helper 的 visible 屬性 in `index.html`
- [x] T044 [US6] 在 `animate()` 中加入 lightHelper.update() 呼叫 in `index.html`

**Checkpoint**: User Story 6 完成 - 使用者可切換輔助視覺化工具

---

## Phase 9: Polish & Cross-Cutting Concerns

**Purpose**: 跨 User Story 的改進和完善

- [x] T045 加入 HTML info 區塊 - 顯示標題和 three.js 連結 in `index.html`
- [x] T046 加入 CSS 全螢幕樣式 - body margin:0, overflow:hidden in `index.html`
- [x] T047 加入 CSS info 區塊樣式 - 絕對定位、置中、白色文字 in `index.html`
- [x] T048 處理紋理載入失敗 - 個別紋理失敗不影響其他紋理 in `index.html`
- [x] T049 處理光源強度為 0 的邊界情況 - 確保場景只剩環境光照明 in `index.html`
- [x] T050 程式碼整理 - 確保程式碼結構清晰、註解完整 in `index.html`
- [x] T051 執行 quickstart.md 驗證 - 確認所有功能符合規格

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1 (Setup)
    │
    ▼
Phase 2 (Foundational) ─── BLOCKS ALL USER STORIES
    │
    ├──▶ Phase 3 (US1: 基礎場景) ◄── MVP
    │         │
    │         ▼
    ├──▶ Phase 4 (US2: 攝影機控制)
    │         │
    │         ▼
    ├──▶ Phase 5 (US3: GUI 光源屬性)
    │         │
    │         ▼
    ├──▶ Phase 6 (US4: 紋理投射)
    │         │
    │         ▼
    ├──▶ Phase 7 (US5: 陰影參數)
    │         │
    │         ▼
    └──▶ Phase 8 (US6: 輔助工具)
              │
              ▼
         Phase 9 (Polish)
```

### User Story Dependencies

- **User Story 1 (P1)**: 基礎場景 - 必須先完成，其他 Story 依賴此場景
- **User Story 2 (P1)**: 攝影機控制 - 依賴 US1 的場景和攝影機
- **User Story 3 (P2)**: GUI 光源屬性 - 依賴 US1 的聚光燈物件
- **User Story 4 (P2)**: 紋理投射 - 依賴 US1 的聚光燈和紋理載入
- **User Story 5 (P3)**: 陰影參數 - 依賴 US1 的聚光燈陰影配置
- **User Story 6 (P3)**: 輔助工具 - 依賴 US1 的聚光燈物件

### Within Each Phase

- 標記 [P] 的任務可並行執行
- 未標記的任務需依序執行
- 每個 User Story 內部: 建立物件 → 配置屬性 → 整合至 init()

---

## Parallel Opportunities

### Phase 2 並行任務

```bash
# 可同時執行:
T006 [P] createRenderer()
T007 [P] createCamera()
T008 [P] loadTextures()
```

### Phase 3 (US1) 並行任務

```bash
# 可同時執行:
T011 [P] 建立 Scene
T012 [P] createGroundPlane()
T013 [P] createFallbackModel()
```

### Phase 5 (US3) 並行任務

```bash
# 可同時執行:
T029 [P] GUI 顏色控制項
T030 [P] GUI 強度滑桿
T031 [P] GUI 距離滑桿
T032 [P] GUI 角度滑桿
T033 [P] GUI 邊緣模糊滑桿
T034 [P] GUI 衰減滑桿
```

### Phase 7 (US5) 並行任務

```bash
# 可同時執行:
T039 [P] GUI 陰影聚焦滑桿
T040 [P] GUI 陰影強度滑桿
```

---

## Implementation Strategy

### MVP First (User Story 1 + 2 Only)

1. 完成 Phase 1: Setup
2. 完成 Phase 2: Foundational (CRITICAL)
3. 完成 Phase 3: User Story 1 (基礎場景)
4. **STOP and VALIDATE**: 測試動態光影效果
5. 完成 Phase 4: User Story 2 (攝影機控制)
6. **DEPLOY/DEMO**: MVP 完成！

### Incremental Delivery

1. Setup + Foundational → 基礎就緒
2. User Story 1 → 測試 → ✅ 動態光影可見
3. User Story 2 → 測試 → ✅ 攝影機可操作
4. User Story 3 → 測試 → ✅ GUI 可調整光源
5. User Story 4 → 測試 → ✅ 紋理可切換
6. User Story 5 → 測試 → ✅ 陰影可調整
7. User Story 6 → 測試 → ✅ 輔助線可切換
8. Polish → 最終驗證

---

## Summary

| 項目 | 數量 |
|------|------|
| **總任務數** | 51 |
| Phase 1 (Setup) | 3 |
| Phase 2 (Foundational) | 7 |
| Phase 3 (US1) | 11 |
| Phase 4 (US2) | 5 |
| Phase 5 (US3) | 9 |
| Phase 6 (US4) | 3 |
| Phase 7 (US5) | 2 |
| Phase 8 (US6) | 4 |
| Phase 9 (Polish) | 7 |
| **可並行任務** | 17 |
| **User Story 任務** | 34 |

### Independent Test Criteria

| User Story | 獨立測試標準 |
|------------|-------------|
| US1 | 開啟頁面即可看到動態光影效果 |
| US2 | 滑鼠拖曳可旋轉視角，滾輪可縮放 |
| US3 | 調整 GUI 參數，光照即時變化 |
| US4 | 切換紋理選項，投射圖案即時改變 |
| US5 | 調整陰影參數，陰影效果即時變化 |
| US6 | 切換 helpers 開關，輔助線顯示/隱藏 |

### Suggested MVP Scope

**MVP = Phase 1 + Phase 2 + Phase 3 + Phase 4**
- 任務數: 3 + 7 + 11 + 5 = **26 任務**
- 交付功能: 完整 3D 場景、動態光影、攝影機互動

````
