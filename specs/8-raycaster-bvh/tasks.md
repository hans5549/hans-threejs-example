# Tasks: WebGL Raycaster BVH 高效能光線投射

**Input**: Design documents from `/specs/8-raycaster-bvh/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/component-interface.md ✅, quickstart.md ✅

**Tests**: 未明確要求測試，本任務清單不包含測試任務

**Organization**: 任務依照使用者故事分組，支援獨立實作與測試

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 可平行執行（不同檔案，無相依性）
- **[Story]**: 所屬使用者故事（US1, US2, US3, US4）
- 包含精確的檔案路徑

## Path Conventions

本專案採用單一 HTML 檔案結構：
- **主檔案**: `examples/webgl-raycaster-bvh/index.html`
- **文件**: `specs/8-raycaster-bvh/`

---

## Phase 1: Setup（專案初始化）

**Purpose**: 建立專案基礎結構與檔案

- [X] T001 建立專案目錄 `examples/webgl-raycaster-bvh/`
- [X] T002 建立基礎 HTML 結構於 `examples/webgl-raycaster-bvh/index.html`，包含 DOCTYPE、meta、title
- [X] T003 [P] 加入 CDN 引用：Three.js r160+、three-mesh-bvh、Stats.js、lil-gui 於 `examples/webgl-raycaster-bvh/index.html`
- [X] T004 [P] 建立基礎 CSS 樣式（全螢幕 canvas、載入進度 UI）於 `examples/webgl-raycaster-bvh/index.html` 的 `<style>` 區塊

---

## Phase 2: Foundational（基礎架構）

**Purpose**: 建立核心 3D 場景基礎設施，所有使用者故事都依賴此階段

**⚠️ CRITICAL**: 此階段必須完成後才能開始任何使用者故事

- [X] T005 實作 `init()` 函式：建立 Scene（background: 0x000000）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T006 實作 PerspectiveCamera 設定（fov: 60, near: 0.1, far: 100, position: (10, 10, 10)）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T007 實作 WebGLRenderer 設定（antialias: true, setPixelRatio）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T008 [P] 實作 OrbitControls 設定（enableDamping: true, dampingFactor: 0.05）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T009 [P] 實作基礎光照系統（AmbientLight + DirectionalLight）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T010 實作 `onWindowResize()` 視窗大小調整處理於 `examples/webgl-raycaster-bvh/index.html`
- [X] T011 實作 `animate()` 主動畫迴圈（requestAnimationFrame）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T012 實作載入狀態管理（LoadingState: idle → loading → building → ready → error）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T013 [P] 實作載入進度 UI（進度條、百分比、狀態訊息）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T014 實作 `loadModel()` 函式：FBXLoader 載入模型，包含 60 秒逾時處理於 `examples/webgl-raycaster-bvh/index.html`
- [X] T015 實作 `showError()` 錯誤訊息顯示函式於 `examples/webgl-raycaster-bvh/index.html`
- [X] T016 擴展 Three.js 原型：`computeBoundsTree`、`disposeBoundsTree`、`acceleratedRaycast` 於 `examples/webgl-raycaster-bvh/index.html`

**Checkpoint**: 基礎架構完成 - 場景可渲染、模型可載入、BVH 擴展已就緒

---

## Phase 3: User Story 1 - 觀察 BVH 加速效果 (Priority: P1) 🎯 MVP

**Goal**: 使用者載入 3D 模型後，能夠清楚看到使用 BVH 加速結構與未使用時的效能差異

**Independent Test**: 載入模型並觀察 FPS 指標，切換 BVH 開關驗證效能差異（啟用: 60+ FPS → 停用: 10-20 FPS）

### Implementation for User Story 1

- [X] T017 [US1] 實作 `buildBVH()` 函式：建構 BVH 加速結構（strategy: CENTER, maxDepth: 40）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T018 [US1] 實作 `toggleBVH()` 函式：啟用/停用 BVH 加速並恢復原始 raycast 於 `examples/webgl-raycaster-bvh/index.html`
- [X] T019 [P] [US1] 實作 `generateRayOrigins()` 函式：Fibonacci 球面採樣產生光線原點（100條, 半徑 5）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T020 [US1] 實作 `castRays()` 函式：執行光線追蹤查詢並收集交點於 `examples/webgl-raycaster-bvh/index.html`
- [X] T021 [P] [US1] 整合 Stats.js：建立 FPS 監控面板於 `examples/webgl-raycaster-bvh/index.html`
- [X] T022 [US1] 實作 PerformanceMonitor：追蹤 FPS 歷史、計算平均值於 `examples/webgl-raycaster-bvh/index.html`
- [X] T023 [US1] 實作 FPS 警告機制：FPS < 30 時顯示效能警告 UI 於 `examples/webgl-raycaster-bvh/index.html`
- [X] T024 [P] [US1] 建立 lil-gui 控制面板基礎結構於 `examples/webgl-raycaster-bvh/index.html`
- [X] T025 [US1] 加入 GUI 控制項：Enable BVH 開關（預設 true）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T026 [US1] 在 animate() 中整合光線追蹤效能計算於 `examples/webgl-raycaster-bvh/index.html`

**Checkpoint**: User Story 1 完成 - 可獨立驗證 BVH 加速效果（60+ FPS vs 10-20 FPS）

---

## Phase 4: User Story 2 - 視覺化光線投射結果 (Priority: P2)

**Goal**: 使用者可以直觀看到多條光線和與模型的相交點

**Independent Test**: 載入場景並觀察螢幕上的光線（白色線條）和相交點（紅色小球體）

### Implementation for User Story 2

- [X] T027 [P] [US2] 建立 RayOrigins InstancedMesh：球體幾何、綠色半透明材質於 `examples/webgl-raycaster-bvh/index.html`
- [X] T028 [P] [US2] 建立 IntersectionPoints InstancedMesh：球體幾何、紅色材質於 `examples/webgl-raycaster-bvh/index.html`
- [X] T029 [US2] 實作 `visualizeRays()` 函式：建立光線 LineSegments 視覺化於 `examples/webgl-raycaster-bvh/index.html`
- [X] T030 [US2] 實作光線材質設定：白色半透明 LineBasicMaterial（opacity: 0.5）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T031 [US2] 實作交點位置更新：更新 IntersectionPoints 實例矩陣於 `examples/webgl-raycaster-bvh/index.html`
- [X] T032 [US2] 實作 `clearRays()` 函式：清除所有光線視覺化元素於 `examples/webgl-raycaster-bvh/index.html`
- [X] T033 [US2] 區分有相交/無相交光線的視覺化（有相交顯示完整光線，無相交延伸到最大距離）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T034 [P] [US2] 加入 GUI 控制項：Ray Opacity 滑桿（範圍 0-1，預設 0.5）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T035 [US2] 實作模型自動旋轉（params.animate: true）於 `examples/webgl-raycaster-bvh/index.html`

**Checkpoint**: User Story 2 完成 - 可觀察光線路徑和相交點視覺化

---

## Phase 5: User Story 3 - 互動式 BVH 視覺化 (Priority: P3)

**Goal**: 使用者可以選擇顯示或隱藏 BVH 樹狀結構的包圍盒視覺化

**Independent Test**: 切換 BVH 視覺化選項並觀察包圍盒的顯示與隱藏

### Implementation for User Story 3

- [X] T036 [US3] 實作 `toggleBVHHelper()` 函式：建立/顯示/隱藏 MeshBVHHelper 於 `examples/webgl-raycaster-bvh/index.html`
- [X] T037 [US3] 實作 `updateBVHHelper()` 函式：更新 BVH 視覺化深度（1-20）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T038 [US3] 設定 BVH Helper 樣式：邊界框顏色、線條寬度於 `examples/webgl-raycaster-bvh/index.html`
- [X] T039 [P] [US3] 加入 GUI 控制項：Show BVH Visualization 開關（預設 false）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T040 [US3] 加入 GUI 控制項：BVH Depth 滑桿（範圍 1-20，預設 10）於 `examples/webgl-raycaster-bvh/index.html`

**Checkpoint**: User Story 3 完成 - 可顯示/隱藏 BVH 層級邊界框

---

## Phase 6: User Story 4 - 控制光線數量 (Priority: P3)

**Goal**: 使用者可以透過控制介面調整同時投射的光線數量

**Independent Test**: 調整光線數量滑桿並觀察 FPS 變化和畫面上光線數量變化

### Implementation for User Story 4

- [X] T041 [US4] 實作 `updateRayCount()` 函式：動態調整光線數量並重新執行光線追蹤於 `examples/webgl-raycaster-bvh/index.html`
- [X] T042 [US4] 實作光線數量為零的處理邏輯（不顯示光線但模型正常顯示）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T043 [P] [US4] 加入 GUI 控制項：Ray Count 滑桿（範圍 1-200，預設 100）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T044 [US4] 實作光線數量變更時的效能影響視覺回饋於 `examples/webgl-raycaster-bvh/index.html`

**Checkpoint**: User Story 4 完成 - 可動態調整光線數量

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: 跨功能優化與最終整合

- [X] T045 [P] 整合所有 GUI 控制項到統一面板（右上角）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T046 [P] 加入 GUI 控制項：Auto Rotate 開關（預設 true）於 `examples/webgl-raycaster-bvh/index.html`
- [X] T047 優化載入流程：載入完成後自動隱藏進度 UI 於 `examples/webgl-raycaster-bvh/index.html`
- [X] T048 實作 WebGL 支援檢測與友善錯誤訊息於 `examples/webgl-raycaster-bvh/index.html`
- [X] T049 [P] 加入頁面標題和基本說明註解於 `examples/webgl-raycaster-bvh/index.html`
- [X] T050 [P] 程式碼清理：移除 console.log、統一命名風格於 `examples/webgl-raycaster-bvh/index.html`
- [X] T051 執行 quickstart.md 驗證流程，確認所有測試場景通過

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 無相依性 - 可立即開始
- **Foundational (Phase 2)**: 依賴 Setup 完成 - **阻擋所有使用者故事**
- **User Story 1 (Phase 3)**: 依賴 Foundational 完成 - 可先完成作為 MVP
- **User Story 2 (Phase 4)**: 依賴 Foundational 完成 - 可與 US1 平行進行
- **User Story 3 (Phase 5)**: 依賴 Foundational 完成 - 需要 BVH 功能（建議在 US1 後）
- **User Story 4 (Phase 6)**: 依賴 Foundational 完成 - 需要光線視覺化（建議在 US2 後）
- **Polish (Phase 7)**: 依賴所有使用者故事完成

### User Story Dependencies

```
Setup (T001-T004)
       │
       ▼
Foundational (T005-T016)
       │
       ├────────────────────────────────────┐
       │                                    │
       ▼                                    ▼
User Story 1 (T017-T026)          User Story 2 (T027-T035)
BVH 加速效果                       光線視覺化
       │                                    │
       ▼                                    ▼
User Story 3 (T036-T040)          User Story 4 (T041-T044)
BVH 視覺化                         光線數量控制
       │                                    │
       └──────────────┬─────────────────────┘
                      │
                      ▼
               Polish (T045-T051)
```

### Parallel Opportunities

**Setup Phase:**
```bash
# 可平行執行:
T003 [P] CDN 引用
T004 [P] CSS 樣式
```

**Foundational Phase:**
```bash
# 可平行執行:
T008 [P] OrbitControls
T009 [P] 光照系統
T013 [P] 載入進度 UI
```

**User Story 1:**
```bash
# 可平行執行:
T019 [P] generateRayOrigins()
T021 [P] Stats.js 整合
T024 [P] lil-gui 基礎結構
```

**User Story 2:**
```bash
# 可平行執行:
T027 [P] RayOrigins InstancedMesh
T028 [P] IntersectionPoints InstancedMesh
T034 [P] Ray Opacity GUI
```

**User Story 3 & 4:**
```bash
# 可平行執行:
T039 [P] BVH Visualization GUI
T043 [P] Ray Count GUI
```

---

## Implementation Strategy

### MVP First (只完成 User Story 1)

1. ✅ Complete Phase 1: Setup (T001-T004)
2. ✅ Complete Phase 2: Foundational (T005-T016)
3. ✅ Complete Phase 3: User Story 1 (T017-T026)
4. **STOP and VALIDATE**: 測試 BVH 加速效果
5. 如果需要可部署展示 MVP

### Incremental Delivery

1. Setup + Foundational → 基礎架構就緒
2. Add User Story 1 → BVH 效能比較功能（MVP!）
3. Add User Story 2 → 光線視覺化（增強展示效果）
4. Add User Story 3 → BVH 結構視覺化（教育功能）
5. Add User Story 4 → 光線數量控制（實驗功能）
6. 每個故事都增加價值而不影響之前的功能

### Single Developer Strategy

建議執行順序：
1. Phase 1 → Phase 2 → Phase 3 (MVP)
2. Phase 4 → Phase 5 → Phase 6
3. Phase 7 (最終整合)

---

## Summary

| 項目 | 數量 |
|------|------|
| **總任務數** | 51 |
| **Setup 任務** | 4 (T001-T004) |
| **Foundational 任務** | 12 (T005-T016) |
| **User Story 1 任務** | 10 (T017-T026) |
| **User Story 2 任務** | 9 (T027-T035) |
| **User Story 3 任務** | 5 (T036-T040) |
| **User Story 4 任務** | 4 (T041-T044) |
| **Polish 任務** | 7 (T045-T051) |
| **可平行任務 [P]** | 18 |
| **MVP 範圍（建議）** | T001-T026 (26 任務) |

---

## Notes

- 所有程式碼都在單一檔案 `examples/webgl-raycaster-bvh/index.html` 中
- [P] 標記的任務可同時進行（寫入不同的程式碼區塊）
- [Story] 標記對應 spec.md 中的使用者故事
- 每個 Checkpoint 都是可驗證的里程碑
- 建議在每個 Phase 完成後進行瀏覽器測試
- 使用 `<!-- Section: XXX -->` 註解來區分程式碼區塊
