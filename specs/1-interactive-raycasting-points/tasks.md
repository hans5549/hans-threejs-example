```markdown
# Tasks: Three.js 互動式點雲 Raycasting 範例

**Input**: Design documents from `/specs/1-interactive-raycasting-points/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, quickstart.md ✅

**Tests**: 未要求自動化測試 - 本範例使用手動瀏覽器測試驗證

**Organization**: 任務按 User Story 分組，以便獨立實作和測試每個故事

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 可平行執行（不同檔案、無相依性）
- **[Story]**: 此任務所屬的 User Story (例如 US1, US2, US3)
- 描述中包含完整檔案路徑

## Path Conventions

根據 plan.md 的專案結構：

```
examples/
└── interactive-raycasting-points/
    └── index.html       # 單一 HTML 檔案 (內嵌 CSS + JS)
```

---

## Phase 1: Setup (基礎建設)

**Purpose**: 專案初始化與基本結構

- [ ] T001 建立專案目錄結構 `examples/interactive-raycasting-points/`
- [ ] T002 建立空白 HTML 骨架檔案 `examples/interactive-raycasting-points/index.html`
- [ ] T003 [P] 配置 Import Map 載入 Three.js r160+ 和 Stats.js 於 `index.html`

---

## Phase 2: Foundational (基礎必要元件)

**Purpose**: 所有 User Story 共享的核心基礎設施

**⚠️ 重要**: 此階段完成前，無法開始任何 User Story 實作

- [ ] T004 實作 CSS 樣式：全螢幕畫布、info 標籤定位、無滾軸 於 `index.html`
- [ ] T005 [P] 實作 Scene 建立與初始化 於 `index.html`
- [ ] T006 [P] 實作 PerspectiveCamera 建立 (fov=45, near=1, far=10000, position=(10,10,10)) 於 `index.html`
- [ ] T007 [P] 實作 WebGLRenderer 建立並附加到 DOM 於 `index.html`
- [ ] T008 實作 window resize 事件處理器：更新 aspect ratio 和 renderer size 於 `index.html`
- [ ] T009 實作動畫迴圈框架 (animate + render functions) 於 `index.html`

**Checkpoint**: 基礎設施就緒 - 可開始 User Story 實作

---

## Phase 3: User Story 1 - 查看 3D 點雲視覺化 (Priority: P1) 🎯 MVP

**Goal**: 使用者開啟網頁後能看到三組不同顏色的 3D 點雲，場景自動緩慢旋轉展示 3D 效果

**Independent Test**: 開啟網頁即可驗證，應看到紅色、綠色、青色三組點雲以波浪形狀排列，場景自動旋轉

### Implementation for User Story 1

- [ ] T010 [P] [US1] 實作 `generatePointCloudGeometry(color, width, length)` 函數：使用 Float32Array 建立 position 和 color attributes 於 `index.html`
- [ ] T011 [P] [US1] 實作波浪形狀數學公式：y = (cos(u×π×4) + sin(v×π×8))/20 於 `index.html`
- [ ] T012 [P] [US1] 實作顏色強度計算：intensity = (y + 0.1) × 5 於 `index.html`
- [ ] T013 [US1] 實作 `generatePointcloud(color, width, length)` 函數：標準點雲建立 於 `index.html`
- [ ] T014 [US1] 實作 `generateIndexedPointcloud(color, width, length)` 函數：帶索引的點雲建立 於 `index.html`
- [ ] T015 [US1] 實作 `generateIndexedWithOffsetPointcloud(color, width, length)` 函數：帶分組偏移的點雲建立 於 `index.html`
- [ ] T016 [US1] 在 init() 中建立三組點雲：紅色(-5,0,0)、綠色(0,0,0)、青色(5,0,0)，scale=(5,10,10) 於 `index.html`
- [ ] T017 [US1] 實作攝影機旋轉：使用 Matrix4.makeRotationY(0.005) 於 render() 中 於 `index.html`

**Checkpoint**: User Story 1 完成，可獨立測試：開啟 HTML 應看到三組彩色點雲自動旋轉

---

## Phase 4: User Story 2 - 滑鼠懸停互動檢測 (Priority: P1)

**Goal**: 使用者移動滑鼠在點雲上方時，系統偵測最近的點並顯示紅色球體標記

**Independent Test**: 移動滑鼠到任一點雲上方，應看到紅色球體標記出現在最接近滑鼠位置的點上

### Implementation for User Story 2

- [ ] T018 [P] [US2] 宣告 Raycaster 和 pointer (Vector2) 全域變數 於 `index.html`
- [ ] T019 [P] [US2] 宣告物件池相關變數：spheres[], spheresIndex, toggle, clock 於 `index.html`
- [ ] T020 [US2] 實作 Raycaster 初始化：設定 params.Points.threshold = 0.1 於 `index.html`
- [ ] T021 [US2] 實作球體物件池：建立 40 個 SphereGeometry(0.1, 32, 32) + MeshBasicMaterial(0xff0000) 於 `index.html`
- [ ] T022 [US2] 實作 `onPointerMove(event)` 事件處理器：轉換滑鼠座標到 NDC 空間 於 `index.html`
- [ ] T023 [US2] 在 render() 中實作 Raycasting：raycaster.setFromCamera + intersectObjects 於 `index.html`
- [ ] T024 [US2] 實作時間節流機制：toggle > 0.02 時才更新標記，使用 clock.getDelta() 於 `index.html`
- [ ] T025 [US2] 實作標記放置邏輯：將球體移動到 intersection.point，重設 scale 為 1 於 `index.html`
- [ ] T026 [US2] 實作球體縮放動畫：每幀 scale *= 0.98，clamp 在 0.01~1 範圍 於 `index.html`

**Checkpoint**: User Stories 1 和 2 完成，點雲顯示且滑鼠互動正常運作

---

## Phase 5: User Story 3 - 效能監控顯示 (Priority: P2)

**Goal**: 使用者能看到即時的 FPS 統計資訊

**Independent Test**: 開啟網頁後，應在畫面左上角看到 FPS 統計面板

### Implementation for User Story 3

- [ ] T027 [P] [US3] Import Stats module 從 'three/addons/libs/stats.module.js' 於 `index.html`
- [ ] T028 [US3] 在 init() 中建立 Stats 並附加到 container 於 `index.html`
- [ ] T029 [US3] 在 animate() 中呼叫 stats.update() 於 `index.html`

**Checkpoint**: User Stories 1, 2, 3 完成，FPS 面板顯示正常

---

## Phase 6: User Story 4 - 響應式視窗調整 (Priority: P3)

**Goal**: 使用者調整視窗大小時，3D 場景自動調整適應

**Independent Test**: 調整瀏覽器視窗大小，場景應正確重新渲染而不變形

### Implementation for User Story 4

- [ ] T030 [US4] 實作 `onWindowResize()` 函數：更新 camera.aspect 和 renderer.setSize 於 `index.html`
- [ ] T031 [US4] 加入 window resize 事件監聽器 於 `index.html`
- [ ] T032 [US4] 實作最小視窗限制：Math.max(window.innerWidth, 320), Math.max(window.innerHeight, 240) 於 `index.html`

**Checkpoint**: 所有 User Stories 完成，視窗調整功能正常

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: 影響多個 User Stories 的改進項目

- [ ] T033 [P] 加入 HTML info 標籤說明文字 於 `index.html`
- [ ] T034 [P] 加入 WebGL 降級偵測：檢查 renderer 是否支援 WebGL 於 `index.html`
- [ ] T035 程式碼整理：確保變數命名一致、移除 console.log 於 `index.html`
- [ ] T036 執行 quickstart.md 驗證：使用 HTTP server 開啟並手動測試所有功能

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 無相依性 - 可立即開始
- **Foundational (Phase 2)**: 相依於 Setup 完成 - **阻擋**所有 user stories
- **User Stories (Phase 3+)**: 全部相依於 Foundational 完成
  - User Story 1 (P1): 基礎設施就緒後可開始
  - User Story 2 (P1): 可與 US1 平行進行（不同程式碼區塊）
  - User Story 3 (P2): 可與 US1/US2 平行進行
  - User Story 4 (P3): 可與其他 stories 平行進行
- **Polish (Final Phase)**: 相依於所有 user stories 完成

### User Story Dependencies

```
Setup (Phase 1)
    │
    ▼
Foundational (Phase 2)  ──────── BLOCKS ALL STORIES
    │
    ├──────────────────┬──────────────────┬──────────────────┐
    │                  │                  │                  │
    ▼                  ▼                  ▼                  ▼
User Story 1       User Story 2      User Story 3      User Story 4
(點雲視覺化)        (滑鼠互動)         (FPS 監控)        (響應式調整)
    │                  │                  │                  │
    └──────────────────┴──────────────────┴──────────────────┘
                                │
                                ▼
                         Polish (Phase 7)
```

### Within Each User Story

- 標記 [P] 的任務可平行執行
- 函數定義優先於函數呼叫
- 核心邏輯優先於整合

### Parallel Opportunities

由於本專案為**單一 HTML 檔案**，平行機會有限但仍可利用：

- **Phase 2**: T005, T006, T007 可平行（不同函數區塊）
- **User Story 1**: T010, T011, T012 可平行（獨立的輔助函數）
- **User Story 2**: T018, T019 可平行（變數宣告）
- **User Story 3**: T027 可與 US2 任務平行

---

## Parallel Example: User Story 1

```bash
# 可平行執行的 US1 任務（不同函數）:
Task: T010 實作 generatePointCloudGeometry() 函數
Task: T011 實作波浪形狀數學公式
Task: T012 實作顏色強度計算

# 需序列執行的 US1 任務（相互依賴）:
Task: T013 generatePointcloud() - 依賴 T010
Task: T014 generateIndexedPointcloud() - 依賴 T010
Task: T015 generateIndexedWithOffsetPointcloud() - 依賴 T010
Task: T016 在 init() 中建立三組點雲 - 依賴 T013, T014, T015
Task: T017 攝影機旋轉邏輯 - 依賴 Phase 2 完成
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. 完成 Phase 1: Setup
2. 完成 Phase 2: Foundational (關鍵 - 阻擋所有 stories)
3. 完成 Phase 3: User Story 1
4. **停止並驗證**: 測試點雲是否正確顯示和旋轉
5. 如果 MVP 足夠 → 部署/展示

### Incremental Delivery

1. 完成 Setup + Foundational → 基礎就緒
2. 新增 User Story 1 → 獨立測試 → 可展示基本視覺化 (MVP!)
3. 新增 User Story 2 → 獨立測試 → 可展示互動功能
4. 新增 User Story 3 → 獨立測試 → 可展示效能監控
5. 新增 User Story 4 → 獨立測試 → 完整功能
6. 每個 story 都增加價值而不破壞之前的功能

### Estimated Time

| Phase | 任務數 | 預估時間 |
|-------|--------|----------|
| Setup | 3 | 5 分鐘 |
| Foundational | 6 | 15 分鐘 |
| User Story 1 | 8 | 30 分鐘 |
| User Story 2 | 9 | 40 分鐘 |
| User Story 3 | 3 | 5 分鐘 |
| User Story 4 | 3 | 10 分鐘 |
| Polish | 4 | 10 分鐘 |
| **Total** | **36** | **~2 小時** |

---

## Notes

- [P] 任務 = 不同檔案或程式碼區塊，無相依性
- [Story] 標籤將任務映射到特定 user story 以便追蹤
- 每個 user story 應可獨立完成和測試
- 每完成一個任務或邏輯群組後提交
- 可在任何 checkpoint 停止並獨立驗證 story
- 避免：模糊任務、同檔案衝突、破壞獨立性的跨 story 相依

```
