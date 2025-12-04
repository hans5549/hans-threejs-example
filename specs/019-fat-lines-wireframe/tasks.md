# Tasks: Fat Lines Wireframe

**Input**: Design documents from `/specs/019-fat-lines-wireframe/`  
**Prerequisites**: plan.md ✓, spec.md ✓, research.md ✓, data-model.md ✓, quickstart.md ✓

**Tests**: 此功能為單一 HTML 檔案的前端展示，不需要自動化測試。使用 quickstart.md 的手動測試方案。

**Organization**: 任務按使用者故事分組，以便獨立實作和測試每個故事。

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: 可平行執行（不同檔案、無相依性）
- **[Story]**: 任務所屬的使用者故事（例如 US1、US2、US3）
- 描述中包含確切的檔案路徑

## Path Conventions

- **Output**: `examples/webgl-lines-fat-wireframe/index.html`
- 單一 HTML 檔案，包含內嵌的 CSS 和 JavaScript

---

## Phase 1: Setup (專案初始化)

**Purpose**: 建立專案目錄結構和基礎 HTML 框架

- [ ] T001 建立目錄結構 `examples/webgl-lines-fat-wireframe/`
- [ ] T002 建立 HTML 基礎結構，包含 DOCTYPE、meta 標籤、標題「three.js webgl - lines - fat - wireframe」於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T003 [P] 設定 import map，配置 three.js r174 及 addons 的 CDN 路徑於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T004 [P] 加入內嵌 CSS 樣式（全螢幕黑色背景、無邊距）於 `examples/webgl-lines-fat-wireframe/index.html`

**Checkpoint**: HTML 框架完成，可在瀏覽器中開啟但尚無 3D 內容

---

## Phase 2: Foundational (核心基礎設施)

**Purpose**: 建立所有使用者故事共用的核心 Three.js 場景、渲染器和動畫迴圈

**⚠️ CRITICAL**: 此階段必須完成後才能開始任何使用者故事

- [ ] T005 實作 ES module script 區塊，匯入所需的 Three.js 模組於 `examples/webgl-lines-fat-wireframe/index.html`
  - THREE 核心模組
  - LineMaterial, Wireframe, WireframeGeometry2 from 'three/addons/lines/'
  - GUI from 'three/addons/libs/lil-gui.module.min.js'
  - Stats from 'three/addons/libs/stats.module.js'
  - OrbitControls from 'three/addons/controls/OrbitControls.js'
- [ ] T006 建立全域變數宣告區塊（camera, camera2, scene, renderer, controls, matLine, stats 等）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T007 實作 `init()` 函式骨架，初始化 WebGLRenderer（antialias: true, alpha: true, clearColor alpha 0）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T008 實作 `animate()` 函式，包含 requestAnimationFrame 迴圈和 render 呼叫於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T009 建立 THREE.Scene 實例於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T010 實作主攝影機 (PerspectiveCamera)，FOV 40、位置 (-50, 0, 50) 於 `examples/webgl-lines-fat-wireframe/index.html`

**Checkpoint**: 基礎場景完成，黑色背景的空白 3D 畫布可正常渲染

---

## Phase 3: User Story 1 - 檢視 Fat Lines Wireframe 3D 模型 (Priority: P1) 🎯 MVP

**Goal**: 使用者開啟頁面後，能看到 Fat Lines 技術渲染的 Icosahedron 線框模型

**Independent Test**: 開啟頁面即可看到藍色粗線條渲染的 3D 線框模型，線條寬度明顯大於 1 像素

### Implementation for User Story 1

- [ ] T011 [US1] 建立 IcosahedronGeometry（radius: 20, detail: 1）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T012 [US1] 建立 WireframeGeometry2 從 IcosahedronGeometry 於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T013 [US1] 建立 LineMaterial（color: 0x4080ff, linewidth: 5）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T014 [US1] 建立 Wireframe 物件並加入場景，呼叫 computeLineDistances() 於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T015 [US1] 在 render 函式中更新 LineMaterial.resolution 為目前視窗大小於 `examples/webgl-lines-fat-wireframe/index.html`

**Checkpoint**: Fat Lines 線框模型可見，線寬為 5 像素，顏色為藍色

---

## Phase 4: User Story 2 - 使用軌道控制旋轉與縮放模型 (Priority: P1) 🎯 MVP

**Goal**: 使用者能透過滑鼠拖曳旋轉模型、滾輪縮放視圖

**Independent Test**: 滑鼠拖曳可旋轉，滾輪可縮放，範圍 10-500

### Implementation for User Story 2

- [ ] T016 [US2] 建立 OrbitControls 並綁定主攝影機和 renderer.domElement 於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T017 [US2] 設定 OrbitControls 縮放限制（minDistance: 10, maxDistance: 500）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T018 [US2] 在 animate 迴圈中呼叫 controls.update() 於 `examples/webgl-lines-fat-wireframe/index.html`

**Checkpoint**: 軌道控制完整運作，使用者可自由旋轉和縮放視圖

---

## Phase 5: User Story 3 - 調整線條寬度 (Priority: P2)

**Goal**: 使用者能透過 GUI 即時調整線寬 1-10 像素

**Independent Test**: 拖曳線寬滑桿可即時看到線條粗細變化

### Implementation for User Story 3

- [ ] T019 [US3] 建立 GUI 實例於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T020 [US3] 定義 GUI 參數物件（params），包含初始值於 `examples/webgl-lines-fat-wireframe/index.html`
  - 'line type': 0
  - 'width (px)': 5
  - 'dashed': false
  - 'dash scale': 1
  - 'dash / gap': 'equal'
- [ ] T021 [US3] 加入 'width (px)' 滑桿控制項（min: 1, max: 10, step: 1）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T022 [US3] 實作 width 變更回呼函式，更新 matLine.linewidth 於 `examples/webgl-lines-fat-wireframe/index.html`

**Checkpoint**: 線寬控制運作，拖曳滑桿可看到即時變化

---

## Phase 6: User Story 4 - 切換虛線模式 (Priority: P2)

**Goal**: 使用者能透過 GUI 開關啟用虛線效果，並調整虛線參數

**Independent Test**: 切換虛線開關可看到線條在實線與虛線之間變化

### Implementation for User Story 4

- [ ] T023 [US4] 加入 'dashed' checkbox 控制項於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T024 [US4] 實作 dashed 變更回呼函式，更新 matLine.dashed、defines.USE_DASH、needsUpdate 於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T025 [US4] 加入 'dash scale' 滑桿控制項（min: 0.5, max: 1, step: 0.1）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T026 [US4] 實作 dashScale 變更回呼函式，更新 matLine.dashScale 於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T027 [US4] 加入 'dash / gap' 選擇器（2:1、equal、1:2）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T028 [US4] 實作 dashGap 變更回呼函式，根據選擇更新 matLine.dashSize 和 matLine.gapSize 於 `examples/webgl-lines-fat-wireframe/index.html`

**Checkpoint**: 虛線控制完整運作，可切換虛線並調整參數

---

## Phase 7: User Story 5 - 比較 Fat Lines 與標準線條 (Priority: P2)

**Goal**: 使用者能在 Fat Lines 和標準 WebGL 線條之間切換

**Independent Test**: 使用類型選擇器切換，可看到渲染方式變化

### Implementation for User Story 5

- [ ] T029 [US5] 建立 WireframeGeometry 從 IcosahedronGeometry（標準版本）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T030 [US5] 建立 LineBasicMaterial（color: 0x4080ff）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T031 [US5] 建立 LineDashedMaterial（color: 0x4080ff, dashSize: 1, gapSize: 1, scale: 2）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T032 [US5] 建立 LineSegments 物件並加入場景，預設 visible: false，呼叫 computeLineDistances() 於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T033 [US5] 加入 'line type' 選擇器（0: LineGeometry, 1: gl.LINE）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T034 [US5] 實作 lineType 變更回呼函式，切換 wireframe.visible 和 wireframe1.visible 於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T035 [US5] 確保切換類型時同步虛線狀態（更新 LineSegments 材質）於 `examples/webgl-lines-fat-wireframe/index.html`

**Checkpoint**: 可在兩種渲染模式之間切換，視覺差異明顯

---

## Phase 8: User Story 6 - 雙視口顯示 (Priority: P3)

**Goal**: 右下角顯示小型插入視口，從第二攝影機角度同步顯示模型

**Independent Test**: 頁面載入後，右下角出現正方形小視口

### Implementation for User Story 6

- [ ] T036 [US6] 建立第二攝影機 camera2（FOV: 40, aspect: 1）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T037 [US6] 在 render 函式中實作雙視口渲染邏輯於 `examples/webgl-lines-fat-wireframe/index.html`
  - 主視口：setViewport(0, 0, width, height)
  - 插入視口：setViewport(width - insetWidth - 16, 16, insetWidth, insetHeight)
- [ ] T038 [US6] 計算插入視口尺寸（高度的 1/4）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T039 [US6] 同步 camera2 位置與旋轉至 camera（或保持獨立視角）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T040 [US6] 啟用 scissorTest 以正確裁切插入視口區域於 `examples/webgl-lines-fat-wireframe/index.html`

**Checkpoint**: 雙視口正確顯示，插入視口位於右下角

---

## Phase 9: Polish & Cross-Cutting Concerns

**Purpose**: 完善響應式設計、效能監控和最終驗證

- [ ] T041 [P] 實作 Stats.js 效能監控（FPS 顯示）於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T042 [P] 實作 window resize 事件處理函式於 `examples/webgl-lines-fat-wireframe/index.html`
  - 更新 renderer.setSize
  - 更新 camera.aspect 和 camera.updateProjectionMatrix
  - 更新插入視口尺寸
- [ ] T043 程式碼清理和格式化於 `examples/webgl-lines-fat-wireframe/index.html`
- [ ] T044 執行 quickstart.md 驗證所有功能於 `examples/webgl-lines-fat-wireframe/index.html`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 無相依性 - 可立即開始
- **Foundational (Phase 2)**: 相依於 Setup 完成 - 阻塞所有使用者故事
- **User Stories (Phase 3-8)**: 全部相依於 Foundational 完成
  - US1 和 US2 (P1) 可先行完成作為 MVP
  - US3-US5 (P2) 需要 GUI 框架（T019-T020）
  - US6 (P3) 可獨立於 GUI 功能
- **Polish (Phase 9)**: 相依於所有使用者故事完成

### User Story Dependencies

- **User Story 1 (P1)**: 無跨故事相依性 - 核心 Fat Lines 渲染
- **User Story 2 (P1)**: 無跨故事相依性 - OrbitControls 獨立運作
- **User Story 3 (P2)**: 相依 T019-T020 (GUI 初始化)
- **User Story 4 (P2)**: 相依 T019-T020 (GUI 初始化)、T013 (LineMaterial 已建立)
- **User Story 5 (P2)**: 相依 T019-T020 (GUI 初始化)、T014 (Fat Lines wireframe)
- **User Story 6 (P3)**: 相依 T010 (主攝影機)，無 GUI 相依性

### Within Each User Story

- 幾何體 → 材質 → 物件建立 → 加入場景
- GUI 控制項 → 回呼函式

### Parallel Opportunities

**可平行執行的任務組：**

1. **Setup Phase**: T003 和 T004 可平行
2. **Phase 9**: T041 和 T042 可平行

**由於單一檔案限制，大部分任務需按順序執行**

---

## Parallel Example: Setup Phase

```bash
# 這些任務操作不同區塊，可平行進行：
Task T003: "設定 import map"
Task T004: "加入內嵌 CSS 樣式"
```

---

## Implementation Strategy

### MVP First (User Story 1 + 2)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational
3. Complete Phase 3: User Story 1 (Fat Lines 渲染)
4. Complete Phase 4: User Story 2 (OrbitControls)
5. **STOP and VALIDATE**: 開啟 index.html 驗證基本功能
6. 可發佈/展示基本版本

### Incremental Delivery

1. Setup + Foundational → 空白場景可渲染
2. Add US1 + US2 → Fat Lines + 控制 (MVP!)
3. Add US3 → 線寬控制
4. Add US4 → 虛線模式
5. Add US5 → 類型切換比較
6. Add US6 → 雙視口
7. Polish → 完整功能

### Task Count Summary

| Phase | Task Count | Cumulative |
|-------|------------|------------|
| Phase 1: Setup | 4 | 4 |
| Phase 2: Foundational | 6 | 10 |
| Phase 3: US1 (P1) | 5 | 15 |
| Phase 4: US2 (P1) | 3 | 18 |
| Phase 5: US3 (P2) | 4 | 22 |
| Phase 6: US4 (P2) | 6 | 28 |
| Phase 7: US5 (P2) | 7 | 35 |
| Phase 8: US6 (P3) | 5 | 40 |
| Phase 9: Polish | 4 | 44 |
| **Total** | **44** | |

---

## Notes

- 所有任務操作同一檔案 `examples/webgl-lines-fat-wireframe/index.html`
- [P] 標記的任務可平行執行（操作不同程式碼區塊）
- [Story] 標籤追蹤任務與使用者故事的對應關係
- 每個使用者故事應能獨立完成和測試
- 每完成一個邏輯群組後提交
- 在任何 checkpoint 停止以獨立驗證故事
- 避免：模糊任務、同區塊衝突、破壞獨立性的跨故事相依性
