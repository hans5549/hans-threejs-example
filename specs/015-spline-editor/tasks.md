````markdown
# Tasks: WebGL Geometry Spline Editor

**Input**: Design documents from `/specs/015-spline-editor/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅, quickstart.md ✅

**Tests**: 無自動化測試（手動瀏覽器驗證）

**Organization**: 任務依 User Story 分組，每個 Story 可獨立實作和測試。

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: 可平行執行（不同程式碼區塊，無相依性）
- **[Story]**: 所屬 User Story（例如 US1, US2, US3）
- 所有任務皆在 `examples/webgl-spline-editor/index.html` 單一檔案中實作

---

## Phase 1: Setup (基礎架構)

**Purpose**: 建立專案結構和檔案基礎

- [ ] T001 建立目錄結構 `examples/webgl-spline-editor/`
- [ ] T002 建立 `examples/webgl-spline-editor/index.html` 骨架，包含 HTML 結構、Import Map 和空白 script module

---

## Phase 2: Foundational (核心基礎設施)

**Purpose**: 所有 User Stories 共用的核心元件，**必須先完成**才能進行 User Story 實作

**⚠️ CRITICAL**: 在此階段完成之前，無法開始任何 User Story 工作

- [ ] T003 實作 Scene、Camera、Renderer 初始化程式碼區塊
- [ ] T004 [P] 實作全域變數宣告區塊（splineHelperObjects, positions, splines, ARC_SEGMENTS 等）
- [ ] T005 [P] 實作 CSS 樣式區塊（body margin:0, overflow:hidden, 全螢幕 canvas）
- [ ] T006 實作 render() 函式，根據 params 設定曲線可見性並呼叫 renderer.render()
- [ ] T007 實作 onWindowResize() 函式，更新 camera aspect 和 renderer size

**Checkpoint**: 基礎架構完成 - 可開始 User Story 實作

---

## Phase 3: User Story 1 - 基本場景與曲線視覺化 (Priority: P1) 🎯 MVP

**Goal**: 使用者開啟頁面後能看到完整的 3D 場景，包含地面、網格、控制點和三種曲線

**Independent Test**: 開啟 `index.html`，確認能看到 3D 場景、地面、網格、4 個彩色立方體控制點和三條不同顏色曲線

### Implementation for User Story 1

- [ ] T008 [US1] 實作光源設置：AmbientLight 和 SpotLight（含陰影設定）
- [ ] T009 [US1] 實作地面 Mesh：PlaneGeometry + ShadowMaterial，y=-200
- [ ] T010 [US1] 實作網格輔助線：GridHelper 2000×100，y=-199，半透明
- [ ] T011 [US1] 實作 addSplineObject(position) 函式：建立控制點立方體 Mesh
- [ ] T012 [US1] 實作預設控制點載入：load() 函式載入 4 個預設座標
- [ ] T013 [US1] 實作 CatmullRomCurve3 曲線建立：三種類型（uniform/centripetal/chordal）
- [ ] T014 [US1] 實作曲線 Line Mesh 建立：BufferGeometry + LineBasicMaterial，三種顏色
- [ ] T015 [US1] 實作 updateSplineOutline() 函式：更新曲線幾何頂點

**Checkpoint**: User Story 1 完成 - 頁面顯示完整 3D 場景、控制點和曲線

---

## Phase 4: User Story 2 - 相機視角控制 (Priority: P1)

**Goal**: 使用者能透過滑鼠控制相機視角（旋轉、縮放、平移）

**Independent Test**: 拖曳滑鼠左鍵旋轉、滾輪縮放、右鍵平移，確認相機控制正常

### Implementation for User Story 2

- [ ] T016 [US2] 實作 OrbitControls 初始化，綁定 camera 和 renderer.domElement
- [ ] T017 [US2] 設定 OrbitControls damping 和 change 事件監聽

**Checkpoint**: User Story 2 完成 - 相機可旋轉、縮放、平移

---

## Phase 5: User Story 3 - 控制點拖曳編輯 (Priority: P1)

**Goal**: 使用者能點擊選取控制點，拖曳移動，曲線即時更新

**Independent Test**: 點擊控制點出現操作手柄，拖曳移動時三條曲線即時更新

### Implementation for User Story 3

- [ ] T018 [US3] 實作 TransformControls 初始化，綁定 camera 和 renderer.domElement
- [ ] T019 [US3] 實作 TransformControls dragging-changed 事件：拖曳時停用 OrbitControls
- [ ] T020 [US3] 實作 TransformControls objectChange 事件：呼叫 updateSplineOutline()
- [ ] T021 [US3] 實作 Raycaster 和 pointer 向量初始化
- [ ] T022 [US3] 實作 onPointerDown/onPointerUp 事件處理：記錄滑鼠位置
- [ ] T023 [US3] 實作點擊選取邏輯：Raycaster 偵測 intersects，attach/detach TransformControls

**Checkpoint**: User Story 3 完成 - 控制點可點擊選取和拖曳，曲線即時更新

---

## Phase 6: User Story 4 - 新增控制點 (Priority: P2)

**Goal**: 使用者能透過 GUI 按鈕新增新控制點

**Independent Test**: 點擊 GUI「Add Point」按鈕，確認新控制點出現且曲線更新

### Implementation for User Story 4

- [ ] T024 [US4] 實作 addPoint() 函式：遞增計數、呼叫 addSplineObject、更新曲線
- [ ] T025 [US4] 在 GUI 中加入「addPoint」按鈕，綁定 addPoint 函式

**Checkpoint**: User Story 4 完成 - 可透過 GUI 新增控制點

---

## Phase 7: User Story 5 - 移除控制點 (Priority: P2)

**Goal**: 使用者能透過 GUI 按鈕移除最後一個控制點（保持最少 4 點）

**Independent Test**: 點擊 GUI「Remove Point」按鈕，確認控制點減少且曲線更新；測試只剩 4 點時無法再移除

### Implementation for User Story 5

- [ ] T026 [US5] 實作 removePoint() 函式：檢查最少 4 點、移除物件、detach TransformControls、更新曲線
- [ ] T027 [US5] 在 GUI 中加入「removePoint」按鈕，綁定 removePoint 函式

**Checkpoint**: User Story 5 完成 - 可透過 GUI 移除控制點，最少保持 4 點

---

## Phase 8: User Story 6 - 曲線類型切換 (Priority: P2)

**Goal**: 使用者能透過 GUI 開關不同類型曲線顯示，調整 uniform 曲線張力

**Independent Test**: 勾選/取消勾選 GUI 選項，確認對應顏色曲線顯示/隱藏；調整 tension 滑桿，確認紅色曲線形狀變化

### Implementation for User Story 6

- [ ] T028 [US6] 實作 params 物件：uniform, tension, centripetal, chordal 屬性
- [ ] T029 [US6] 在 GUI 中加入 uniform checkbox，onChange 呼叫 render()
- [ ] T030 [US6] 在 GUI 中加入 tension slider (0-1, step 0.01)，onChange 更新曲線張力並重繪
- [ ] T031 [US6] 在 GUI 中加入 centripetal checkbox，onChange 呼叫 render()
- [ ] T032 [US6] 在 GUI 中加入 chordal checkbox，onChange 呼叫 render()

**Checkpoint**: User Story 6 完成 - 可切換曲線顯示和調整張力

---

## Phase 9: User Story 7 - 匯出曲線資料 (Priority: P3)

**Goal**: 使用者能透過 GUI 按鈕匯出控制點座標到主控台

**Independent Test**: 點擊 GUI「Export」按鈕，開啟瀏覽器 DevTools Console 確認輸出 Vector3 陣列

### Implementation for User Story 7

- [ ] T033 [US7] 實作 exportSpline() 函式：遍歷控制點、格式化為 Vector3 字串、console.log 輸出
- [ ] T034 [US7] 在 GUI 中加入「exportSpline」按鈕，綁定 exportSpline 函式

**Checkpoint**: User Story 7 完成 - 可匯出控制點座標到主控台

---

## Phase 10: Polish & Cross-Cutting Concerns

**Purpose**: 最終整合和優化

- [ ] T035 [P] 加入 HTML title 和 meta viewport 標籤
- [ ] T036 整合所有事件監聽器：pointerdown, pointerup, resize
- [ ] T037 實作 init() 函式呼叫順序整合和 window.onload 初始化
- [ ] T038 執行 quickstart.md 驗證：測試所有 7 個 User Story 功能
- [ ] T039 [P] 程式碼整理：確保變數宣告順序、函式順序符合相依性

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 無相依性 - 可立即開始
- **Foundational (Phase 2)**: 依賴 Setup 完成 - **阻塞所有 User Stories**
- **User Stories (Phase 3-9)**: 全部依賴 Foundational 完成
  - US1 (P1): 基礎視覺化，無 Story 相依
  - US2 (P1): 依賴 US1 完成（需要場景存在）
  - US3 (P1): 依賴 US1、US2 完成（需要控制點和 OrbitControls）
  - US4 (P2): 依賴 US1 完成
  - US5 (P2): 依賴 US1、US4 完成（需要 addSplineObject 存在）
  - US6 (P2): 依賴 US1 完成（需要曲線存在）
  - US7 (P3): 依賴 US1 完成（需要控制點存在）
- **Polish (Phase 10)**: 依賴所有 User Stories 完成

### User Story Dependencies Graph

```text
      Phase 2 (Foundational)
              │
              ▼
    ┌─────────┴─────────┐
    │                   │
    ▼                   ▼
  US1 ───────────────► US2 ───────────────► US3
    │                                         │
    │                                         │
    ├─────────────► US4 ─────────────────────┘
    │                │
    ├─────────────► US5 (依賴 US4 的 addSplineObject)
    │
    ├─────────────► US6
    │
    └─────────────► US7
```

### Within Each User Story

- 先完成模型/資料結構
- 再完成函式實作
- 最後完成 GUI 整合
- Story 完成後再進入下一優先級

### Parallel Opportunities

- T004, T005 可與 T003 平行執行
- T008, T009, T010 可平行執行（不同場景元件）
- T021, T022, T023 需依序執行（Raycaster 流程）
- T035, T039 可與其他 Polish 任務平行

---

## Parallel Example: Phase 3 (US1)

```bash
# 光源和地面可平行建立：
Task: "實作光源設置：AmbientLight 和 SpotLight"
Task: "實作地面 Mesh：PlaneGeometry + ShadowMaterial"
Task: "實作網格輔助線：GridHelper"

# 控制點和曲線需依序：
Task: "實作 addSplineObject(position) 函式"
Task: "實作預設控制點載入：load() 函式"
Task: "實作 CatmullRomCurve3 曲線建立" # 需要控制點位置
Task: "實作曲線 Line Mesh 建立"
Task: "實作 updateSplineOutline() 函式"
```

---

## Implementation Strategy

### MVP First (Phase 1-3: Setup + Foundational + US1)

1. 完成 Phase 1: Setup - 建立檔案結構
2. 完成 Phase 2: Foundational - 基礎架構
3. 完成 Phase 3: User Story 1 - 場景視覺化
4. **STOP and VALIDATE**: 開啟瀏覽器測試 US1

### Incremental Delivery

1. Setup + Foundational → 基礎架構就緒
2. + US1 → 測試 → 可展示（MVP！）
3. + US2 → 測試 → 相機控制
4. + US3 → 測試 → 核心編輯功能完成
5. + US4, US5 → 測試 → 控制點管理
6. + US6 → 測試 → 曲線參數控制
7. + US7 → 測試 → 匯出功能
8. + Polish → 最終驗證

### Estimated Effort

| Phase | 任務數 | 預估時間 |
|-------|--------|----------|
| Setup | 2 | 5 min |
| Foundational | 5 | 15 min |
| US1 | 8 | 30 min |
| US2 | 2 | 10 min |
| US3 | 6 | 25 min |
| US4 | 2 | 10 min |
| US5 | 2 | 10 min |
| US6 | 5 | 15 min |
| US7 | 2 | 10 min |
| Polish | 5 | 15 min |
| **Total** | **39** | **~2.5 hours** |

---

## Notes

- 所有任務皆在單一 `index.html` 檔案中實作
- [P] 標記的任務可平行執行（不同程式碼區塊）
- [Story] 標記對應 spec.md 中的 User Story
- 每個 Story 完成後應獨立可測試
- 無自動化測試，以瀏覽器手動驗證為主
- 參考 research.md 中的程式碼範例實作

````
