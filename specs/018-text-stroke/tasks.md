# Tasks: WebGL 文字描邊效果

**Feature Branch**: `018-text-stroke`  
**Input**: Design documents from `/specs/018-text-stroke/`  
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Tests**: 本專案為視覺展示範例，採用手動視覺測試，不包含自動化測試任務。

**Organization**: 任務按 User Story 組織，以實現每個故事的獨立實作和測試。

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 可平行執行（不同檔案、無相依性）
- **[Story]**: 對應的 User Story（US1, US2, US3, US4）
- 描述中包含確切的檔案路徑

## Path Conventions

本專案採用單一 HTML 檔案結構：
- **範例檔案**: `examples/webgl-geometry-text-stroke/index.html`
- **字體資源**: `fonts/MPLUSRounded1c/MPLUSRounded1c-Regular.typeface.json.zip`

---

## Phase 1: Setup (基礎架構)

**Purpose**: 建立專案基本結構和 Three.js 場景

- [ ] T001 建立範例目錄結構 `examples/webgl-geometry-text-stroke/`
- [ ] T002 建立 HTML 檔案基本結構，包含 `<!DOCTYPE html>`、`<head>`、`<body>` 在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T003 設定 importmap 引用 Three.js 0.170.0 及相關模組（Font, SVGLoader, OrbitControls, fflate）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T004 [P] 新增基本 CSS 樣式（body margin:0, canvas display:block）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T005 [P] 下載字體檔案 `MPLUSRounded1c-Regular.typeface.json.zip` 至 `fonts/MPLUSRounded1c/` 目錄

**Checkpoint**: 基本 HTML 結構完成，可在瀏覽器開啟但尚無任何渲染

---

## Phase 2: Foundational (Three.js 核心初始化)

**Purpose**: 建立 Three.js 場景、相機、渲染器等核心元件，這些是所有 User Story 的基礎

**⚠️ CRITICAL**: 必須完成此階段才能開始任何 User Story 實作

- [ ] T006 建立 `init()` 函式骨架在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T007 建立 PerspectiveCamera（fov: 45, near: 1, far: 10000）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T008 [P] 建立 Scene 並設定深色背景（0x000000）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T009 建立 WebGLRenderer（antialias: true）並加入 DOM 在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T010 設定相機初始位置（z: 350）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T011 建立 `render()` 函式和 animation loop 在 `examples/webgl-geometry-text-stroke/index.html`

**Checkpoint**: 可看到黑色背景的空白 WebGL 畫布

---

## Phase 3: User Story 1 - 觀看 3D 文字描邊效果 (Priority: P1) 🎯 MVP

**Goal**: 展示具有半透明填充和實心描邊的 3D 文字

**Independent Test**: 打開頁面即可看到完整渲染的描邊文字，無需操作

**Maps to**: FR-001, FR-002, FR-003, FR-004, FR-006, FR-007, FR-008, SC-001, SC-005

### 字體載入實作

- [ ] T012 [US1] 建立 loadFont() 函式使用 FileLoader 載入 zip 檔案在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T013 [US1] 實作 fflate 解壓縮邏輯（unzipSync, strFromU8）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T014 [US1] 解析 JSON 並建立 Font 物件在 `examples/webgl-geometry-text-stroke/index.html`

### 材質建立

- [ ] T015 [P] [US1] 建立 color 變數（THREE.Color, 0x006699）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T016 [P] [US1] 建立 matDark（MeshBasicMaterial, 實心）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T017 [P] [US1] 建立 matLite（MeshBasicMaterial, 半透明 opacity: 0.4）在 `examples/webgl-geometry-text-stroke/index.html`

### 核心描邊生成函式

- [ ] T018 [US1] 建立 generateStrokeText(text, font, size, direction, material) 函式簽名在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T019 [US1] 實作 font.generateShapes(text, size, direction) 取得形狀陣列在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T020 [US1] 建立 ShapeGeometry 作為填充幾何體在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T021 [US1] 計算 bounding box 並置中對齊填充幾何體在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T022 [US1] 建立填充 Mesh 並加入 Group 在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T023 [US1] 遍歷 shapes 提取所有 holes 加入形狀陣列在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T024 [US1] 使用 SVGLoader.getStrokeStyle 建立描邊樣式在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T025 [US1] 為每個形狀呼叫 SVGLoader.pointsToStroke 建立描邊幾何體在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T026 [US1] 建立描邊 Mesh 並置中對齊、加入 Group 在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T027 [US1] 回傳完整的 Group 物件在 `examples/webgl-geometry-text-stroke/index.html`

### 初始文字渲染

- [ ] T028 [US1] 在 loadFont 回呼中呼叫 generateStrokeText 建立第一組文字在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T029 [US1] 將文字 Group 加入 Scene 在 `examples/webgl-geometry-text-stroke/index.html`

**Checkpoint**: 頁面顯示一組具有半透明填充和實心描邊的文字，User Story 1 可獨立測試驗證

---

## Phase 4: User Story 2 - 瀏覽多語言多方向文字 (Priority: P1)

**Goal**: 展示三種不同書寫方向的文字（ltr, rtl, tb）

**Independent Test**: 觀察場景中三組不同方向排列的文字是否正確顯示

**Maps to**: FR-005, SC-002

### 多方向文字實作

- [ ] T030 [US2] 建立英文文字 "Three.js"（direction: 'ltr'）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T031 [US2] 定位英文文字位置（y: 100）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T032 [US2] 建立希伯來文文字 "שָׁלוֹם"（direction: 'rtl'）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T033 [US2] 定位希伯來文文字位置（y: 0）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T034 [US2] 建立中文文字 "中文"（direction: 'tb'）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T035 [US2] 定位中文文字位置（y: -100 或合適位置）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T036 [US2] 將三組文字 Group 全部加入 Scene 在 `examples/webgl-geometry-text-stroke/index.html`

**Checkpoint**: 頁面同時顯示三組不同方向的描邊文字，User Story 2 可獨立測試驗證

---

## Phase 5: User Story 3 - 使用軌道控制器檢視文字 (Priority: P2)

**Goal**: 提供滑鼠互動讓使用者旋轉、縮放、平移視角

**Independent Test**: 使用滑鼠拖曳、滾輪、右鍵拖曳測試三種控制方式

**Maps to**: FR-009, SC-003

### OrbitControls 實作

- [ ] T037 [US3] 引入 OrbitControls（已在 importmap 設定）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T038 [US3] 建立 OrbitControls 實例並綁定 camera 和 renderer.domElement 在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T039 [US3] 在 render 函式中呼叫 controls.update() 在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T040 [US3] 測試旋轉功能（左鍵拖曳）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T041 [US3] 測試縮放功能（滾輪）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T042 [US3] 測試平移功能（右鍵拖曳）在 `examples/webgl-geometry-text-stroke/index.html`

**Checkpoint**: 可透過滑鼠互動從任意角度觀察 3D 文字，User Story 3 可獨立測試驗證

---

## Phase 6: User Story 4 - 響應式視窗調整 (Priority: P3)

**Goal**: 視窗大小改變時自動調整場景

**Independent Test**: 調整瀏覽器視窗大小，觀察場景是否正確重新渲染

**Maps to**: FR-010, SC-004

### 響應式實作

- [ ] T043 [US4] 建立 onWindowResize() 函式在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T044 [US4] 更新 camera.aspect 為新的視窗比例在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T045 [US4] 呼叫 camera.updateProjectionMatrix() 在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T046 [US4] 呼叫 renderer.setSize(window.innerWidth, window.innerHeight) 在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T047 [US4] 註冊 window 'resize' 事件監聽器在 `examples/webgl-geometry-text-stroke/index.html`

**Checkpoint**: 視窗調整後場景正確重新渲染，User Story 4 可獨立測試驗證

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: 最終調整和品質確認

- [ ] T048 [P] 驗證所有功能需求（FR-001 至 FR-010）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T049 [P] 驗證所有成功標準（SC-001 至 SC-005）在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T050 檢查 Edge Cases：字體載入失敗時的錯誤處理在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T051 [P] 程式碼清理和註解完善在 `examples/webgl-geometry-text-stroke/index.html`
- [ ] T052 執行 quickstart.md 驗證流程

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 無相依性 - 可立即開始
- **Foundational (Phase 2)**: 相依於 Setup 完成 - 阻塞所有 User Stories
- **User Story 1 (Phase 3)**: 相依於 Foundational 完成 - 核心 MVP
- **User Story 2 (Phase 4)**: 相依於 User Story 1 完成（使用 generateStrokeText 函式）
- **User Story 3 (Phase 5)**: 相依於 Foundational 完成 - 可與 US1/US2 平行
- **User Story 4 (Phase 6)**: 相依於 Foundational 完成 - 可與 US1/US2/US3 平行
- **Polish (Phase 7)**: 相依於所有 User Stories 完成

### User Story Dependencies

```
Setup (P1) ──┬──> Foundational (P2) ──┬──> US1 (P3) ──> US2 (P4)
             │                        │
             │                        ├──> US3 (P5) ─────┐
             │                        │                  │
             │                        └──> US4 (P6) ─────┼──> Polish (P7)
             │                                           │
             └─────────────────────────────────────────────┘
```

### Within Each User Story

- 材質建立可平行執行（T015, T016, T017）
- 字體載入必須先完成才能建立文字
- 核心函式 generateStrokeText 必須完成才能建立多組文字

### Parallel Opportunities

**Phase 1 平行任務**:
- T004 和 T005 可同時進行

**Phase 2 平行任務**:
- T008 可與其他任務平行

**Phase 3 平行任務**:
- T015, T016, T017（材質建立）可同時進行

**跨 User Story 平行**:
- 一旦 Foundational 完成，US3 和 US4 可與 US1/US2 平行進行

---

## Parallel Example: User Story 1 Materials

```bash
# 同時建立所有材質（可平行執行）:
Task T015: 建立 color 變數（THREE.Color, 0x006699）
Task T016: 建立 matDark（MeshBasicMaterial, 實心）
Task T017: 建立 matLite（MeshBasicMaterial, 半透明）
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. 完成 Phase 1: Setup
2. 完成 Phase 2: Foundational
3. 完成 Phase 3: User Story 1
4. **STOP and VALIDATE**: 頁面顯示具有描邊效果的文字
5. 可選擇此時部署/展示 MVP

### Incremental Delivery

1. 完成 Setup + Foundational → 基礎架構就緒
2. 加入 User Story 1 → 獨立測試 → MVP 完成！
3. 加入 User Story 2 → 獨立測試 → 多方向文字
4. 加入 User Story 3 → 獨立測試 → 互動控制
5. 加入 User Story 4 → 獨立測試 → 響應式完成
6. 每個 Story 都增加價值但不破壞之前的功能

### Single Developer Strategy

由於本專案為單一 HTML 檔案，建議順序執行：
1. Phase 1 → Phase 2 → Phase 3 (MVP)
2. Phase 4 → Phase 5 → Phase 6
3. Phase 7 (收尾)

---

## Notes

- [P] 標記的任務 = 不同區塊、無相依性，可平行執行
- [Story] 標籤將任務對應到特定 User Story 以便追蹤
- 每個 User Story 應可獨立完成和測試
- 每個任務或邏輯群組完成後提交
- 在任何 Checkpoint 停止以獨立驗證 Story
- 避免：模糊的任務、同檔案衝突、破壞獨立性的跨 Story 相依

---

## Summary

| 項目 | 數量 |
|------|------|
| 總任務數 | 52 |
| Phase 1 (Setup) | 5 |
| Phase 2 (Foundational) | 6 |
| Phase 3 (US1 - MVP) | 18 |
| Phase 4 (US2) | 7 |
| Phase 5 (US3) | 6 |
| Phase 6 (US4) | 5 |
| Phase 7 (Polish) | 5 |
| 可平行任務 | 15 |
| MVP 範圍 | Phase 1-3 (29 tasks) |
