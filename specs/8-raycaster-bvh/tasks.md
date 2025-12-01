# Tasks: WebGL Raycaster BVH 高效能光線投射

**Input**: Design documents from `/specs/8-raycaster-bvh/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/component-interface.md, quickstart.md

**Tests**: This feature does NOT require automated tests - validation will be done through manual browser testing and performance comparison as specified in quickstart.md.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

---

## Format: `- [ ] [ID] [P?] [Story?] Description with file path`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3, US4)
- Include exact file paths in descriptions

---

## Phase 1: Setup (專案初始化)

**Purpose**: 建立基本專案結構和 HTML 檔案框架

- [X] T001 建立專案目錄結構 examples/webgl-raycaster-bvh/
- [X] T002 建立 index.html 主檔案 examples/webgl-raycaster-bvh/index.html
- [X] T003 [P] 在 index.html 中加入基本 HTML 結構 (doctype, head, body)
- [X] T004 [P] 在 index.html 中加入 CDN 載入: Three.js r160+, three-mesh-bvh v0.7.3, Stats.js, lil-gui

---

## Phase 2: Foundational (核心基礎建設)

**Purpose**: 場景初始化、載入系統、BVH 整合 - 所有使用者故事的前置需求

**⚠️ CRITICAL**: 此階段必須完成才能開始任何使用者故事實作

- [X] T005 實作 init() 函式: 建立 THREE.Scene, PerspectiveCamera, WebGLRenderer in examples/webgl-raycaster-bvh/index.html
- [X] T006 [P] 實作 onWindowResize() 函式處理視窗大小調整 in examples/webgl-raycaster-bvh/index.html
- [X] T007 [P] 實作 OrbitControls 整合讓使用者旋轉和縮放場景 in examples/webgl-raycaster-bvh/index.html
- [X] T008 實作 LoadingState 狀態管理物件 (status, progress, message, startTime) in examples/webgl-raycaster-bvh/index.html
- [X] T009 實作 loadModel() 函式: 使用 FBXLoader 和 LoadingManager 載入 Stanford Bunny in examples/webgl-raycaster-bvh/index.html
- [X] T010 實作載入進度指示器 UI (顯示百分比或動畫) in examples/webgl-raycaster-bvh/index.html
- [X] T011 實作 60 秒載入逾時機制 (Promise.race) in examples/webgl-raycaster-bvh/index.html
- [X] T012 實作 showError() 函式處理載入失敗和逾時錯誤 in examples/webgl-raycaster-bvh/index.html
- [X] T013 擴展 BufferGeometry.prototype 加入 computeBoundsTree, disposeBoundsTree, acceleratedRaycast in examples/webgl-raycaster-bvh/index.html
- [X] T014 實作 buildBVH() 函式: 使用 three-mesh-bvh 建構 BVH (strategy: CENTER, maxDepth: 40, maxLeafTris: 10) in examples/webgl-raycaster-bvh/index.html
- [X] T015 實作 disposeBVH() 函式清除 BVH 加速結構 in examples/webgl-raycaster-bvh/index.html
- [X] T016 實作 animate() 主迴圈: requestAnimationFrame, renderer.render() in examples/webgl-raycaster-bvh/index.html
- [X] T017 加入場景光照 (AmbientLight, DirectionalLight) 和基本材質設定 in examples/webgl-raycaster-bvh/index.html

**Checkpoint**: 基礎完成 - 可載入模型、建構 BVH、顯示場景，使用者故事實作可並行開始

---

## Phase 3: User Story 1 - 觀察 BVH 加速效果 (Priority: P1) 🎯 MVP

**Goal**: 使用者能看到啟用/停用 BVH 時的明顯效能差異 (60 FPS vs 10-20 FPS)

**Independent Test**: 載入模型後觀察 Stats.js FPS 指標，切換 BVH 開關驗證效能差異達 70%

### Implementation for User Story 1

- [X] T018 [P] [US1] 整合 Stats.js: 建立 stats 物件並加入 DOM in examples/webgl-raycaster-bvh/index.html
- [X] T019 [P] [US1] 建立 PerformanceMonitor 類別: 追蹤 FPS 歷史 (60 幀滾動視窗) in examples/webgl-raycaster-bvh/index.html
- [X] T020 [US1] 實作 updatePerformanceMetrics() 函式: 計算平均 FPS 和幀時間 in examples/webgl-raycaster-bvh/index.html
- [X] T021 [US1] 實作 checkPerformanceWarning() 函式: FPS < 30 時顯示警告訊息 in examples/webgl-raycaster-bvh/index.html
- [X] T022 [US1] 實作 toggleBVH() 函式: 啟用/停用 BVH 加速並更新 Raycaster in examples/webgl-raycaster-bvh/index.html
- [X] T023 [US1] 建立 GUI 控制介面: 加入 BVH 開關 (enableBVH checkbox) in examples/webgl-raycaster-bvh/index.html
- [X] T024 [US1] 在 animate() 迴圈中更新 Stats.js 和 PerformanceMonitor in examples/webgl-raycaster-bvh/index.html
- [X] T025 [US1] 實作效能警告 UI 顯示邏輯 (動態警告訊息) in examples/webgl-raycaster-bvh/index.html

**Checkpoint**: 使用者故事 1 完成 - 可觀察 BVH 效能差異並獨立測試

---

## Phase 4: User Story 2 - 視覺化光線投射結果 (Priority: P2)

**Goal**: 使用者能看到從隨機位置投射的光線和相交點

**Independent Test**: 載入場景後觀察螢幕上的半透明光線和相交點小球體

### Implementation for User Story 2

- [X] T026 [P] [US2] 實作 generateRayOrigins() 函式: Fibonacci 球面取樣產生 100 個隨機位置 in examples/webgl-raycaster-bvh/index.html
- [X] T027 [P] [US2] 建立 Ray Origins InstancedMesh: 100 個小球體標記光線起點 in examples/webgl-raycaster-bvh/index.html
- [X] T028 [US2] 實作 castRays() 函式: 使用 Raycaster 偵測相交並回傳 Intersection[] in examples/webgl-raycaster-bvh/index.html
- [X] T029 [US2] 建立 Intersection Points InstancedMesh: 動態數量的小球體標記相交點 in examples/webgl-raycaster-bvh/index.html
- [X] T030 [US2] 實作 visualizeRays() 函式: 使用 LineSegments 顯示光線路徑 in examples/webgl-raycaster-bvh/index.html
- [X] T031 [US2] 實作 updateRayVisualization() 函式: 更新光線和相交點的位置矩陣 in examples/webgl-raycaster-bvh/index.html
- [X] T032 [US2] 實作 clearRays() 函式: 清除場景中的舊光線和相交點 in examples/webgl-raycaster-bvh/index.html
- [X] T033 [US2] 在 animate() 迴圈中呼叫 castRays() 和 updateRayVisualization() in examples/webgl-raycaster-bvh/index.html
- [X] T034 [US2] 設定光線材質為半透明 (opacity: 0.5, transparent: true) in examples/webgl-raycaster-bvh/index.html

**Checkpoint**: 使用者故事 1 和 2 都能獨立運作

---

## Phase 5: User Story 3 - 互動式 BVH 視覺化 (Priority: P3)

**Goal**: 使用者能切換顯示/隱藏 BVH 包圍盒視覺化

**Independent Test**: 切換 BVH 視覺化選項觀察包圍盒框線的顯示與隱藏

### Implementation for User Story 3

- [X] T035 [P] [US3] 整合 MeshBVHHelper: 建立 helper 物件並加入場景 in examples/webgl-raycaster-bvh/index.html
- [X] T036 [P] [US3] 實作 toggleBVHHelper() 函式: 顯示/隱藏 BVH 包圍盒視覺化 in examples/webgl-raycaster-bvh/index.html
- [X] T037 [US3] 實作 updateBVHHelper() 函式: 調整顯示深度 (1-20) in examples/webgl-raycaster-bvh/index.html
- [X] T038 [US3] 加入 GUI 控制: showBVHHelper checkbox 和 helperDepth slider (1-20) in examples/webgl-raycaster-bvh/index.html
- [X] T039 [US3] 實作 BVH Helper 的動態更新邏輯 (深度改變時重建) in examples/webgl-raycaster-bvh/index.html

**Checkpoint**: 使用者故事 1, 2 和 3 都能獨立運作

---

## Phase 6: User Story 4 - 控制光線數量 (Priority: P3)

**Goal**: 使用者能調整光線數量並觀察效能影響

**Independent Test**: 調整光線數量滑桿並觀察 FPS 變化和畫面上的光線數量

### Implementation for User Story 4

- [X] T040 [P] [US4] 實作 updateRayCount() 函式: 根據新數量重新產生光線 (1-200 範圍) in examples/webgl-raycaster-bvh/index.html
- [X] T041 [P] [US4] 實作 GUIParams 物件: 集中管理所有 GUI 參數 (rayCount, enableBVH, showBVHHelper, helperDepth, rayOpacity, animate) in examples/webgl-raycaster-bvh/index.html
- [X] T042 [US4] 加入 GUI 控制: rayCount slider (1-200, step: 10, default: 100) in examples/webgl-raycaster-bvh/index.html
- [X] T043 [US4] 實作光線數量變更回呼: 清除舊光線、重新產生、更新視覺化 in examples/webgl-raycaster-bvh/index.html
- [X] T044 [US4] 加入 GUI 控制: rayOpacity slider (0.1-1.0, step: 0.1, default: 0.5) in examples/webgl-raycaster-bvh/index.html
- [X] T045 [US4] 實作光線透明度動態更新邏輯 in examples/webgl-raycaster-bvh/index.html
- [X] T046 [US4] 加入 GUI 控制: animate checkbox 控制模型自動旋轉 in examples/webgl-raycaster-bvh/index.html

**Checkpoint**: 所有使用者故事獨立運作正常

---

## Phase 7: Polish & Cross-Cutting Concerns (完善與跨領域改進)

**Purpose**: 最終優化和完善

- [ ] T047 [P] 優化記憶體使用: 實作物件池 (object pooling) 重用 InstancedMesh 實例 in examples/webgl-raycaster-bvh/index.html
- [ ] T048 [P] 加入詳細錯誤處理: 分類錯誤類型 (ModelLoadError, BVHBuildError, TimeoutError) in examples/webgl-raycaster-bvh/index.html
- [ ] T049 [P] 優化光線視覺化效能: 使用 setDrawRange 減少不必要的渲染 in examples/webgl-raycaster-bvh/index.html
- [ ] T050 [P] 加入 WebGL 支援檢測: 顯示友善的不支援訊息 in examples/webgl-raycaster-bvh/index.html
- [ ] T051 程式碼重構: 將核心邏輯分離成獨立函式 (提高可讀性) in examples/webgl-raycaster-bvh/index.html
- [X] T052 [P] 加入程式碼註解: 解釋關鍵演算法和決策 in examples/webgl-raycaster-bvh/index.html
- [X] T053 [P] 加入 CSS 樣式: 美化 loading UI, error messages, performance warning in examples/webgl-raycaster-bvh/index.html
- [ ] T054 依據 quickstart.md 執行完整驗證測試 (3 項性能測試)
- [ ] T055 驗證所有 8 項成功標準 (SC-001 to SC-008) from spec.md

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 無相依性 - 可立即開始
- **Foundational (Phase 2)**: 依賴 Setup 完成 - **阻塞所有使用者故事**
- **User Stories (Phase 3-6)**: 所有故事依賴 Foundational 完成
  - 故事可並行實作 (如有足夠人力)
  - 或依優先順序循序實作 (P1 → P2 → P3 → P3)
- **Polish (Phase 7)**: 依賴所有所需使用者故事完成

### User Story Dependencies

- **User Story 1 (P1)**: Foundational 完成後可開始 - 不依賴其他故事
- **User Story 2 (P2)**: Foundational 完成後可開始 - 需整合 US1 的 BVH 切換功能
- **User Story 3 (P3)**: Foundational 完成後可開始 - 需整合 US1 的 BVH 基礎建設
- **User Story 4 (P3)**: Foundational 完成後可開始 - 需整合 US2 的光線產生和視覺化

### Within Each User Story

- GUI 控制優先於互動邏輯
- 核心功能優先於視覺優化
- 完成故事後才進入下個優先級

### Parallel Opportunities

- Setup 階段: T003 和 T004 可並行
- Foundational 階段: T006, T007 可在 T005 完成後並行
- User Story 1: T018, T019 可並行開始
- User Story 2: T026, T027 可並行開始
- User Story 3: T035, T036 可並行開始
- User Story 4: T040, T041 可並行開始
- Polish 階段: 所有標記 [P] 的任務可並行

---

## Parallel Example: User Story 1 Implementation

```bash
# Terminal 1: 實作效能監控
開始 T018 (Stats.js 整合)
開始 T019 (PerformanceMonitor 類別)

# Terminal 2: 等待 T018, T019 完成後
開始 T020 (updatePerformanceMetrics)
開始 T021 (checkPerformanceWarning)

# Terminal 3: 等待基礎完成後
開始 T022 (toggleBVH)
開始 T023 (GUI 控制介面)

# Terminal 4: 整合所有組件
開始 T024 (animate 迴圈更新)
開始 T025 (警告 UI)
```

---

## Implementation Strategy

### MVP First (Minimum Viable Product)

建議以 **User Story 1** 作為 MVP:
- 證明 BVH 加速效果 (核心價值)
- 包含基礎場景、模型載入、效能監控
- 可獨立展示和驗證
- 完成 MVP 後再擴展 US2, US3, US4

### Incremental Delivery

1. **Sprint 1**: Setup + Foundational (T001-T017) - 建立可載入模型的基礎場景
2. **Sprint 2**: User Story 1 (T018-T025) - MVP 可展示 BVH 效能差異
3. **Sprint 3**: User Story 2 (T026-T034) - 加入光線視覺化
4. **Sprint 4**: User Story 3 & 4 (T035-T046) - 進階控制功能
5. **Sprint 5**: Polish (T047-T055) - 最終優化和驗證

### Quality Gates

- **每個 Phase 完成後**: 執行手動測試確認功能正常
- **每個 User Story 完成後**: 執行該故事的獨立測試場景
- **Phase 7 開始前**: 所有功能需求 (FR-001 to FR-018) 必須完成
- **最終交付前**: 所有成功標準 (SC-001 to SC-008) 必須達成

---

## Task Count Summary

- **Total Tasks**: 55
- **Setup (Phase 1)**: 4 tasks
- **Foundational (Phase 2)**: 13 tasks
- **User Story 1 (Phase 3)**: 8 tasks
- **User Story 2 (Phase 4)**: 9 tasks
- **User Story 3 (Phase 5)**: 5 tasks
- **User Story 4 (Phase 6)**: 7 tasks
- **Polish (Phase 7)**: 9 tasks
- **Parallel Opportunities**: 18 tasks marked [P]
- **MVP Scope**: Phase 1 + Phase 2 + Phase 3 (25 tasks)

---

## Format Validation ✅

All tasks follow the required checklist format:
- ✅ All tasks start with `- [ ]` (checkbox)
- ✅ All tasks have sequential Task IDs (T001-T055)
- ✅ All user story tasks have [Story] labels ([US1], [US2], [US3], [US4])
- ✅ Setup and Foundational phases have NO story labels
- ✅ Polish phase has NO story labels
- ✅ All parallelizable tasks marked with [P]
- ✅ All descriptions include exact file paths

---

Generated: 2025-12-01
Based on: spec.md, plan.md, research.md, data-model.md, contracts/component-interface.md, quickstart.md
