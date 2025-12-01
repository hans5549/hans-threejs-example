# Tasks: WebGPU Custom Lighting Model

**Input**: Design documents from `/specs/13-webgpu-lights-custom/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Tests**: 此範例為前端展示頁面，不包含自動化測試，使用手動瀏覽器驗證。

**Organization**: 任務按使用者故事組織，以支援獨立實作和測試。

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: 可平行執行（不同檔案，無相依性）
- **[Story]**: 所屬使用者故事（US1, US2, US3）
- 包含精確檔案路徑

## Path Conventions

- **專案類型**: Single HTML file
- **範例目錄**: `examples/webgpu-lights-custom/`
- **主檔案**: `examples/webgpu-lights-custom/index.html`

---

## Phase 1: Setup (共用基礎架構)

**Purpose**: 專案初始化和基本結構建立

- [X] T001 建立範例目錄結構 `examples/webgpu-lights-custom/` 和 `examples/webgpu-lights-custom/assets/`
- [X] T002 建立基本 HTML 結構 `examples/webgpu-lights-custom/index.html`（doctype, head, body）
- [X] T003 [P] 設定 ES Modules import map（three, three/webgpu, three/tsl, three/addons/）在 `examples/webgpu-lights-custom/index.html`
- [X] T004 [P] 建立 WebGPU 不支援時的 fallback 圖片 `examples/webgpu-lights-custom/assets/fallback.svg`

---

## Phase 2: Foundational (阻斷性前置作業)

**Purpose**: 所有使用者故事開始前必須完成的核心基礎設施

**⚠️ 關鍵**: 此階段完成前無法開始任何使用者故事

- [X] T005 實作 WebGPU 支援偵測邏輯（檢查 navigator.gpu）在 `examples/webgpu-lights-custom/index.html`
- [X] T006 建立 fallback UI 元素（圖片容器 + 說明文字）在 `examples/webgpu-lights-custom/index.html`
- [X] T007 [P] 建立 CSS 基礎樣式（body, 全螢幕, 黑色背景）在 `examples/webgpu-lights-custom/index.html`
- [X] T008 [P] 建立 loading 動畫 UI 元素（spinner + 文字）在 `examples/webgpu-lights-custom/index.html`
- [X] T009 [P] 建立 info header UI 元素（標題 + 說明）在 `examples/webgpu-lights-custom/index.html`
- [X] T010 初始化 WebGPURenderer（antialias: true, setPixelRatio, setSize）在 `examples/webgpu-lights-custom/index.html`
- [X] T011 建立 PerspectiveCamera（FOV: 70, near: 0.1, far: 10, position.z: 1.5）在 `examples/webgpu-lights-custom/index.html`
- [X] T012 建立 Scene 物件在 `examples/webgpu-lights-custom/index.html`
- [X] T013 建立 setAnimationLoop 動畫迴圈框架在 `examples/webgpu-lights-custom/index.html`

**Checkpoint**: Foundation ready - 使用者故事實作現在可以開始

---

## Phase 3: User Story 1 - 觀看動態光源點雲效果 (Priority: P1) 🎯 MVP

**Goal**: 使用者開啟網頁後能看到 50 萬粒子點雲被三個動態彩色光源照亮

**Independent Test**: 開啟網頁即可看到動態光照效果，無需任何使用者操作

### Implementation for User Story 1

- [X] T014 [P] [US1] 建立 CustomLightingModel 類別繼承 THREE.LightingModel 在 `examples/webgpu-lights-custom/index.html`
- [X] T015 [P] [US1] 建立共用 SphereGeometry（radius: 0.02, widthSegments: 16, heightSegments: 8）在 `examples/webgpu-lights-custom/index.html`
- [X] T016 [US1] 實作 CustomLightingModel.direct() 方法（lightColor 加到 directDiffuse）在 `examples/webgpu-lights-custom/index.html`
- [X] T017 [US1] 建立 addLight(hexColor) 工廠函式在 `examples/webgpu-lights-custom/index.html`
- [X] T018 [US1] 在 addLight 中建立 NodeMaterial 自發光材質（colorNode + 空 lightsNode）在 `examples/webgpu-lights-custom/index.html`
- [X] T019 [US1] 在 addLight 中建立 PointLight（intensity: 0.1, distance: 1）並附加 Mesh 在 `examples/webgpu-lights-custom/index.html`
- [X] T020 [US1] 建立三個 PointLight（light1: 0xffaa00, light2: 0x0040ff, light3: 0x80ff80）在 `examples/webgpu-lights-custom/index.html`
- [X] T021 [US1] 建立選擇性光源節點 lights([light1, light2, light3]) 在 `examples/webgpu-lights-custom/index.html`
- [X] T022 [US1] 建立光照模型上下文 allLightsNode.context({ lightingModel }) 在 `examples/webgpu-lights-custom/index.html`
- [X] T023 [US1] 產生 50 萬個隨機 Vector3 點（範圍 ±1.5）在 `examples/webgpu-lights-custom/index.html`
- [X] T024 [US1] 建立 BufferGeometry.setFromPoints(points) 在 `examples/webgpu-lights-custom/index.html`
- [X] T025 [US1] 建立 PointsNodeMaterial 並設定 lightsNode 為光照模型上下文在 `examples/webgpu-lights-custom/index.html`
- [X] T026 [US1] 建立 Points 物件並加入 scene 在 `examples/webgpu-lights-custom/index.html`
- [X] T027 [US1] 在 animate() 中實作光源位置更新（使用三角函數，scale: 0.5）在 `examples/webgpu-lights-custom/index.html`
- [X] T028 [US1] 為三個光源設定不同頻率參數（0.3, 0.5, 0.7）在 `examples/webgpu-lights-custom/index.html`
- [X] T029 [US1] 在 animate() 中實作場景緩慢旋轉（scene.rotation.y = time * 0.1）在 `examples/webgpu-lights-custom/index.html`
- [X] T030 [US1] 在 animate() 中呼叫 renderer.render(scene, camera) 在 `examples/webgpu-lights-custom/index.html`

**Checkpoint**: User Story 1 應已完整可用且可獨立測試

---

## Phase 4: User Story 2 - 互動式場景導覽 (Priority: P2)

**Goal**: 使用者能透過滑鼠拖曳旋轉視角、滾輪縮放距離

**Independent Test**: 使用滑鼠操作即可驗證視角控制功能

### Implementation for User Story 2

- [X] T031 [US2] 匯入 OrbitControls from 'three/addons/controls/OrbitControls.js' 在 `examples/webgpu-lights-custom/index.html`
- [X] T032 [US2] 建立 OrbitControls 實例（camera, renderer.domElement）在 `examples/webgpu-lights-custom/index.html`
- [X] T033 [US2] 設定 OrbitControls.minDistance = 0 在 `examples/webgpu-lights-custom/index.html`
- [X] T034 [US2] 設定 OrbitControls.maxDistance = 4 在 `examples/webgpu-lights-custom/index.html`

**Checkpoint**: User Stories 1 AND 2 應皆可獨立運作

---

## Phase 5: User Story 3 - 響應式視窗調整 (Priority: P3)

**Goal**: 調整瀏覽器視窗大小時畫面自動調整並保持正確比例

**Independent Test**: 調整瀏覽器視窗大小即可驗證

### Implementation for User Story 3

- [X] T035 [US3] 實作 onWindowResize() 函式在 `examples/webgpu-lights-custom/index.html`
- [X] T036 [US3] 在 onWindowResize 中更新 camera.aspect 和呼叫 updateProjectionMatrix() 在 `examples/webgpu-lights-custom/index.html`
- [X] T037 [US3] 在 onWindowResize 中呼叫 renderer.setSize(window.innerWidth, window.innerHeight) 在 `examples/webgpu-lights-custom/index.html`
- [X] T038 [US3] 綁定 window resize 事件到 onWindowResize 在 `examples/webgpu-lights-custom/index.html`

**Checkpoint**: 所有使用者故事現在應皆可獨立運作

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: 影響多個使用者故事的改進

- [X] T039 [P] 在場景就緒時隱藏 loading 動畫在 `examples/webgpu-lights-custom/index.html`
- [X] T040 [P] 完善 CSS 樣式（info 區域、loading 動畫、fallback 區域）在 `examples/webgpu-lights-custom/index.html`
- [ ] T041 驗證 SC-001：頁面載入後 3 秒內呈現完整的動態光照效果
- [ ] T042 驗證 SC-002：點雲包含 50 萬粒子且動畫流暢（目標 60 FPS）
- [ ] T043 驗證 SC-003：三個光源同時可見且各自以不同軌跡移動
- [ ] T044 驗證 SC-004：使用者可在 360 度範圍內自由旋轉視角
- [ ] T045 驗證 SC-005：視窗調整大小後畫面自動適應
- [ ] T046 驗證 SC-006：光源接近粒子時顏色變化清晰可見
- [ ] T047 執行 quickstart.md 驗證流程

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 無相依性 - 可立即開始
- **Foundational (Phase 2)**: 相依於 Setup 完成 - 阻斷所有使用者故事
- **User Stories (Phase 3+)**: 皆相依於 Foundational 階段完成
  - 使用者故事可平行進行（若有人力）
  - 或按優先順序依序進行（P1 → P2 → P3）
- **Polish (Phase 6)**: 相依於所有目標使用者故事完成

### User Story Dependencies

- **User Story 1 (P1)**: Foundational 完成後可開始 - 不相依其他故事
- **User Story 2 (P2)**: Foundational 完成後可開始 - 與 US1 平行但需場景存在
- **User Story 3 (P3)**: Foundational 完成後可開始 - 與 US1/US2 平行但需 renderer 和 camera 存在

### Within Each User Story

- 模型/類別先於服務/函式
- 核心實作先於整合
- 故事完成後再進行下一優先級

### Parallel Opportunities

- Phase 1: T003, T004 可平行
- Phase 2: T007, T008, T009 可平行
- Phase 3 (US1): T014, T015 可平行
- Phase 6: T039, T040 可平行

---

## Parallel Example: User Story 1

```bash
# 平行啟動 User Story 1 的初始任務：
Task: "T014 建立 CustomLightingModel 類別"
Task: "T015 建立共用 SphereGeometry"

# 接續（需等待 T014 完成）：
Task: "T016 實作 CustomLightingModel.direct() 方法"
```

---

## Implementation Strategy

### MVP First (僅 User Story 1)

1. 完成 Phase 1: Setup
2. 完成 Phase 2: Foundational（關鍵 - 阻斷所有故事）
3. 完成 Phase 3: User Story 1
4. **停止並驗證**: 獨立測試 User Story 1
5. 若就緒則部署/展示

### Incremental Delivery

1. 完成 Setup + Foundational → 基礎就緒
2. 加入 User Story 1 → 獨立測試 → 部署/展示 (MVP!)
3. 加入 User Story 2 → 獨立測試 → 部署/展示
4. 加入 User Story 3 → 獨立測試 → 部署/展示
5. 每個故事增加價值而不破壞先前的故事

---

## Notes

- 所有任務在同一檔案 `examples/webgpu-lights-custom/index.html` 執行
- 由於單檔案結構，實際平行度有限，但任務順序仍依相依性排列
- [P] 標記的任務理論上可平行，但實際需注意檔案衝突
- 每個任務或邏輯群組完成後提交
- 在任何檢查點可停止以獨立驗證故事
