# Tasks: WebGL BufferGeometry Lines

**Input**: Design documents from `/specs/11-buffergeometry-lines/`
**Prerequisites**: plan.md ✓, spec.md ✓, research.md ✓, data-model.md ✓, contracts/ ✓

**Tests**: 無自動化測試 - 使用手動瀏覽器測試

**Organization**: 任務依使用者情境分組，每個情境可獨立實作與測試。

---

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: 可平行執行（不同檔案、無相依性）
- **[Story]**: 所屬使用者情境 (US1, US2, US3, US4)
- 包含確切檔案路徑

---

## Phase 1: Setup (專案初始化)

**Purpose**: 建立專案目錄結構和基礎檔案

- [x] T001 建立專案目錄 examples/webgl-buffergeometry-lines/
- [x] T002 [P] 建立 fallback 圖片佔位符 examples/webgl-buffergeometry-lines/fallback.svg
- [x] T003 建立 HTML 基礎結構和樣式 examples/webgl-buffergeometry-lines/index.html

---

## Phase 2: Foundational (基礎架構)

**Purpose**: 建立所有使用者情境共用的核心基礎設施

**⚠️ CRITICAL**: 必須完成此階段才能開始任何使用者情境的實作

- [x] T004 設定 import map 和 Three.js/Stats 模組匯入 examples/webgl-buffergeometry-lines/index.html
- [x] T005 宣告全域變數和常數（container, camera, scene, renderer, line, segments, r, t）examples/webgl-buffergeometry-lines/index.html
- [x] T006 實作 init() 函式骨架（含 try-catch 和 WebGL 錯誤處理）examples/webgl-buffergeometry-lines/index.html
- [x] T007 實作 showFallback() 函式 examples/webgl-buffergeometry-lines/index.html
- [x] T008 建立 WebGL Fallback 樣式 (.fallback) examples/webgl-buffergeometry-lines/index.html

**Checkpoint**: 基礎架構就緒 - 可開始使用者情境的實作 ✅

---

## Phase 3: User Story 1 - 瀏覽 3D 線段視覺化 (Priority: P1) 🎯 MVP

**Goal**: 使用者開啟頁面後能看到 10,000 條彩色線段在 3D 空間中旋轉

**Independent Test**: 開啟頁面，確認顯示彩色線段並持續旋轉

### Implementation for User Story 1

- [x] T009 [US1] 建立透視相機 (PerspectiveCamera) 並設定 FOV=27, position.z=2750 examples/webgl-buffergeometry-lines/index.html
- [x] T010 [US1] 建立場景 (Scene) 和 Timer，連接 document examples/webgl-buffergeometry-lines/index.html
- [x] T011 [US1] 建立 BufferGeometry 並產生 10,000 個隨機頂點位置 examples/webgl-buffergeometry-lines/index.html
- [x] T012 [US1] 計算並設定頂點顏色（根據位置的 RGB 漸層公式）examples/webgl-buffergeometry-lines/index.html
- [x] T013 [US1] 設定 position 和 color 的 Float32BufferAttribute examples/webgl-buffergeometry-lines/index.html
- [x] T014 [US1] 建立 LineBasicMaterial (vertexColors: true) examples/webgl-buffergeometry-lines/index.html
- [x] T015 [US1] 建立 Line 物件並加入場景，計算 boundingSphere examples/webgl-buffergeometry-lines/index.html
- [x] T016 [US1] 建立 WebGLRenderer 並設定尺寸、pixel ratio examples/webgl-buffergeometry-lines/index.html
- [x] T017 [US1] 實作 animate() 函式 - Timer 更新和旋轉動畫 (rotation.x/y) examples/webgl-buffergeometry-lines/index.html
- [x] T018 [US1] 設定 renderer.setAnimationLoop(animate) 啟動動畫迴圈 examples/webgl-buffergeometry-lines/index.html

**Checkpoint**: User Story 1 完成 - 可看到彩色線段旋轉（MVP 可展示）✅

---

## Phase 4: User Story 2 - Morph Target 動畫效果 (Priority: P2)

**Goal**: 線段除了旋轉外，還會週期性變形

**Independent Test**: 觀察線段形態是否隨時間週期性變化

### Implementation for User Story 2

- [x] T019 [US2] 實作 generateMorphTargets(geometry) 函式 examples/webgl-buffergeometry-lines/index.html
- [x] T020 [US2] 產生 morph target 的隨機位置資料陣列 examples/webgl-buffergeometry-lines/index.html
- [x] T021 [US2] 建立 Float32BufferAttribute 並設定 morphTarget.name examples/webgl-buffergeometry-lines/index.html
- [x] T022 [US2] 將 morphTarget 加入 geometry.morphAttributes.position examples/webgl-buffergeometry-lines/index.html
- [x] T023 [US2] 在 init() 中呼叫 generateMorphTargets() examples/webgl-buffergeometry-lines/index.html
- [x] T024 [US2] 在 animate() 中更新 morph 時間變數 t (t += delta * 0.5) examples/webgl-buffergeometry-lines/index.html
- [x] T025 [US2] 在 animate() 中更新 morphTargetInfluences[0] = abs(sin(t)) examples/webgl-buffergeometry-lines/index.html

**Checkpoint**: User Story 2 完成 - 線段有旋轉 + 形態變換動畫 ✅

---

## Phase 5: User Story 3 - 效能統計 (Priority: P3)

**Goal**: 左上角顯示即時 FPS 統計面板

**Independent Test**: 確認頁面左上角有 Stats 面板且數值即時更新

### Implementation for User Story 3

- [x] T026 [US3] 宣告 stats 全域變數 examples/webgl-buffergeometry-lines/index.html
- [x] T027 [US3] 在 init() 中建立 Stats 實例並加入 DOM examples/webgl-buffergeometry-lines/index.html
- [x] T028 [US3] 在 animate() 結尾呼叫 stats.update() examples/webgl-buffergeometry-lines/index.html

**Checkpoint**: User Story 3 完成 - FPS 面板顯示並即時更新 ✅

---

## Phase 6: User Story 4 - 響應式視窗調整 (Priority: P3)

**Goal**: 視窗大小改變時場景自動適配

**Independent Test**: 調整瀏覽器視窗大小，確認場景正確縮放無變形

### Implementation for User Story 4

- [x] T029 [US4] 實作 onWindowResize() 函式 examples/webgl-buffergeometry-lines/index.html
- [x] T030 [US4] 更新 camera.aspect 和呼叫 updateProjectionMatrix() examples/webgl-buffergeometry-lines/index.html
- [x] T031 [US4] 更新 renderer.setSize() examples/webgl-buffergeometry-lines/index.html
- [x] T032 [US4] 在 init() 中註冊 window resize 事件監聽器 examples/webgl-buffergeometry-lines/index.html

**Checkpoint**: User Story 4 完成 - 視窗調整時場景正確適配 ✅

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: 最終驗證和品質確保

- [x] T033 [P] 建立實際的 fallback.svg 圖片 examples/webgl-buffergeometry-lines/fallback.svg
- [ ] T034 驗證 WebGL 不支援時的 fallback 顯示正確
- [ ] T035 驗證效能達標（≥30 FPS @ 10,000 頂點）
- [ ] T036 驗證頁面載入時間 ≤3 秒
- [x] T037 [P] 程式碼清理和註解
- [ ] T038 執行 quickstart.md 驗證流程

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1: Setup ──────────────────┐
                                 ▼
Phase 2: Foundational ───────────┤ (BLOCKS all user stories)
                                 │
         ┌───────────────────────┼───────────────────────┐
         ▼                       ▼                       ▼
Phase 3: US1 (P1)          Phase 4: US2 (P2)      Phase 5: US3 (P3)
    MVP 🎯                       │                       │
         │                       │                       │
         ├───────────────────────┤                       │
         │                       ▼                       │
         │               Phase 6: US4 (P3)               │
         │                       │                       │
         └───────────────────────┴───────────────────────┘
                                 │
                                 ▼
                         Phase 7: Polish
```

### User Story Dependencies

| User Story | Depends On | Can Run In Parallel With |
|------------|------------|--------------------------|
| US1 (P1) | Phase 2 (Foundational) | - |
| US2 (P2) | US1 完成 (需要 geometry 和 animate) | US3, US4 |
| US3 (P3) | Phase 2 (Foundational) | US2, US4 |
| US4 (P3) | US1 完成 (需要 camera 和 renderer) | US2, US3 |

### Within Each User Story

1. 依序執行任務（同一檔案，需保持程式碼一致性）
2. 完成後執行 Independent Test 驗證
3. 驗證通過後可進入下一個 User Story

### Parallel Opportunities

由於所有程式碼都在同一個 `index.html` 檔案中，任務本身需依序執行。但可以：

- **T002 (fallback.png)** 與 **T003-T032** 平行準備
- **T033 (實際 fallback 圖片)** 與其他 Polish 任務平行
- **US3** 可在 **US1** 完成後與 **US2**、**US4** 平行開發（如果有多人團隊）

---

## Implementation Strategy

### MVP First (僅 User Story 1)

1. 完成 Phase 1: Setup
2. 完成 Phase 2: Foundational (CRITICAL)
3. 完成 Phase 3: User Story 1
4. **STOP and VALIDATE**: 測試線段是否正確渲染和旋轉
5. 可展示/部署 MVP

### Incremental Delivery

1. Setup + Foundational → 基礎就緒
2. + User Story 1 → 獨立測試 → **MVP 可展示！**
3. + User Story 2 → 獨立測試 → Morph 動畫完成
4. + User Story 3 → 獨立測試 → FPS 監控完成
5. + User Story 4 → 獨立測試 → 響應式完成
6. Polish → 品質驗證 → **完整版本！**

---

## Notes

- 所有任務都在同一個 `index.html` 檔案中，需依序執行以避免衝突
- [P] 標記的任務涉及不同檔案，可真正平行執行
- [USx] 標記將任務映射到特定使用者情境以便追蹤
- 每個 User Story 都應可獨立完成和測試
- 在每個 Checkpoint 處停下來驗證該 Story 是否獨立運作
- 完成每個任務或邏輯群組後進行 commit
