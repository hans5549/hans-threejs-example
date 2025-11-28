# Tasks: Unreal Bloom Post-processing Effect

**Input**: Design documents from `/specs/2-unreal-bloom/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, quickstart.md ✅

**Tests**: 本功能為前端視覺化展示，使用手動視覺測試驗證。無自動化測試需求。

**Organization**: 任務按使用者故事分組，支援獨立實作和測試。

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: 可平行執行（不同檔案，無依賴）
- **[Story]**: 該任務所屬的使用者故事（如 US1、US2、US3、US4）
- 描述包含精確的檔案路徑

## Path Conventions

本專案採用單一 HTML 檔案架構：
```
examples/unreal-bloom-postprocessing/
└── index.html    # 主要應用程式檔案
```

---

## Phase 1: Setup（基礎建設）

**Purpose**: 專案初始化與基本結構

- [ ] T001 建立專案目錄結構 `examples/unreal-bloom-postprocessing/`
- [ ] T002 建立 HTML 基本架構與 meta 標籤 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T003 設定 importmap 從 CDN 載入 Three.js 與所有 addons in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T004 [P] 建立基本 CSS 樣式（全螢幕 canvas、info 區塊）in `examples/unreal-bloom-postprocessing/index.html`

**Checkpoint**: 專案結構就緒，可開始實作核心功能

---

## Phase 2: Foundational（阻塞性前置任務）

**Purpose**: 所有使用者故事都依賴的核心基礎設施

**⚠️ CRITICAL**: 此階段必須完成後才能開始任何使用者故事

- [ ] T005 建立 JavaScript module 骨架（import 語句、init 函式、animate 函式）in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T006 實作 Scene 建立與背景設定 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T007 實作 PerspectiveCamera 建立與初始位置設定 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T008 實作燈光系統（AmbientLight + PointLight attached to camera）in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T009 實作 WebGLRenderer 建立與基本設定 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T010 實作 Clock 物件用於動畫時間追蹤 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T011 實作 WebGL 支援偵測與 2D 降級版本（顯示說明文字）in `examples/unreal-bloom-postprocessing/index.html`

**Checkpoint**: 基礎渲染架構就緒，可開始使用者故事實作

---

## Phase 3: User Story 1 & 2 - Bloom 效果與視角控制 (Priority: P1) 🎯 MVP

**Goal**: 使用者能看到帶有 Bloom 發光效果的動畫 3D 模型，並使用滑鼠旋轉視角觀察

**Independent Test**: 
1. 開啟網頁確認 3D 模型顯示且有明顯光暈效果
2. 拖曳滑鼠確認視角可旋轉
3. 滾動滑鼠滾輪確認可縮放

### 3D 模型載入與動畫（US1 核心）

- [ ] T012 [US1] 實作 GLTFLoader 載入 PrimaryIonDrive.glb 模型（含錯誤處理與重試按鈕）in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T013 [US1] 實作 AnimationMixer 播放模型內建動畫 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T014 [US1] 在 animate 函式中更新 mixer.update(delta) in `examples/unreal-bloom-postprocessing/index.html`

### 後處理管線（US1 核心）

- [ ] T015 [US1] 實作 EffectComposer 建立與設定 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T016 [US1] 實作 RenderPass 加入 composer in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T017 [US1] 實作 UnrealBloomPass 建立與參數設定（resolution, strength=1.5, radius=0.4, threshold=0.85）in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T018 [US1] 實作 OutputPass 加入 composer in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T019 [US1] 修改 animate 函式使用 composer.render() 取代 renderer.render() in `examples/unreal-bloom-postprocessing/index.html`

### 相機控制（US2 核心）

- [ ] T020 [US2] 實作 OrbitControls 建立與綁定 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T021 [US2] 設定相機垂直旋轉限制 maxPolarAngle = Math.PI * 0.5 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T022 [US2] 設定相機縮放距離限制 minDistance=3, maxDistance=8 in `examples/unreal-bloom-postprocessing/index.html`

### 效能監控

- [ ] T023 [P] [US1] 實作 Stats 效能計數器顯示 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T024 [US1] 在 animate 函式中更新 stats.update() in `examples/unreal-bloom-postprocessing/index.html`

**Checkpoint**: MVP 完成 - 使用者可看到 Bloom 效果並自由旋轉視角觀察模型

---

## Phase 4: User Story 3 - GUI 參數調整 (Priority: P2)

**Goal**: 使用者可透過 GUI 控制面板即時調整 Bloom 效果參數

**Independent Test**: 調整 GUI 中任一滑桿，確認畫面效果立即更新

### GUI 實作

- [ ] T025 [US3] 建立 params 物件（threshold=0, strength=1, radius=0.5, exposure=1）in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T026 [US3] 實作 GUI 建立與 bloom 資料夾 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T027 [US3] 實作 threshold 滑桿（0.0-1.0）與 onChange 回呼更新 bloomPass.threshold in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T028 [US3] 實作 strength 滑桿（0.0-3.0）與 onChange 回呼更新 bloomPass.strength in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T029 [US3] 實作 radius 滑桿（0.0-1.0, step=0.01）與 onChange 回呼更新 bloomPass.radius in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T030 [US3] 實作 tone mapping 資料夾與 exposure 滑桿（0.1-2.0）更新 renderer.toneMappingExposure in `examples/unreal-bloom-postprocessing/index.html`

**Checkpoint**: GUI 功能完成 - 使用者可即時調整所有 Bloom 參數

---

## Phase 5: User Story 4 - 響應式視窗調整 (Priority: P3)

**Goal**: 視窗大小改變時，3D 場景自動適應新尺寸

**Independent Test**: 調整瀏覽器視窗大小，確認畫面比例正確且 Bloom 效果正常

### 響應式實作

- [ ] T031 [US4] 實作 onWindowResize 函式更新 camera.aspect 與 camera.updateProjectionMatrix() in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T032 [US4] 在 onWindowResize 中更新 renderer.setSize() in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T033 [US4] 在 onWindowResize 中更新 composer.setSize() in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T034 [US4] 註冊 window.addEventListener('resize', onWindowResize) in `examples/unreal-bloom-postprocessing/index.html`

**Checkpoint**: 響應式功能完成 - 所有使用者故事皆已獨立可用

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: 跨功能改善與最終優化

- [ ] T035 [P] 新增 HTML info 區塊顯示專案說明與作者資訊 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T036 程式碼整理與註解優化 in `examples/unreal-bloom-postprocessing/index.html`
- [ ] T037 執行 quickstart.md 驗證流程確認所有功能正常運作
- [ ] T038 [P] 更新 README.md 新增此範例的連結與說明

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 無依賴 - 可立即開始
- **Foundational (Phase 2)**: 依賴 Setup 完成 - 阻塞所有使用者故事
- **User Stories (Phase 3-5)**: 依賴 Foundational 完成
  - US1 + US2 (Phase 3): 可立即開始（MVP 核心）
  - US3 (Phase 4): 可在 Phase 3 之後或平行開始
  - US4 (Phase 5): 可在 Phase 3 之後或平行開始
- **Polish (Phase 6)**: 依賴所有使用者故事完成

### User Story Dependencies

- **User Story 1 (P1)**: Foundational 完成後可開始 - 無跨故事依賴
- **User Story 2 (P1)**: Foundational 完成後可開始 - 與 US1 整合但獨立可測
- **User Story 3 (P2)**: 需要 US1 的 bloomPass 物件存在
- **User Story 4 (P3)**: 需要 US1 的 renderer、composer、camera 物件存在

### Within Each User Story

- 依序完成各任務
- 每個 Checkpoint 驗證功能獨立可用
- 完成一個故事後再進行下一個優先級

### Parallel Opportunities

- Setup 階段：T001-T004 可平行
- Foundational 階段：大部分任務有依賴，T011 可與其他平行
- Phase 3 內：T023 可與其他平行
- Phase 4 內：T027-T030 可平行（不同 GUI 元素）
- Phase 5 內：順序執行（同一函式內邏輯）
- Phase 6：T035、T038 可平行

---

## Parallel Example: Phase 3

```bash
# 平行執行不同 Pass 的建立（雖然最終要依序 addPass，但可平行準備程式碼）
# 實際上由於單檔案架構，建議依序完成以避免合併衝突

# 可平行執行：
Task T023: "實作 Stats 效能計數器顯示"
Task T020: "實作 OrbitControls 建立與綁定"
# 因為它們操作不同的變數和程式碼區塊
```

---

## Implementation Strategy

### MVP First（僅 User Story 1 & 2）

1. 完成 Phase 1: Setup
2. 完成 Phase 2: Foundational（關鍵 - 阻塞所有故事）
3. 完成 Phase 3: User Story 1 & 2
4. **停止並驗證**: 獨立測試 MVP 功能
5. 若準備好可部署/展示

### Incremental Delivery

1. 完成 Setup + Foundational → 基礎就緒
2. 新增 User Story 1 & 2 → 獨立測試 → 部署/展示（MVP！）
3. 新增 User Story 3 → 獨立測試 → 部署/展示
4. 新增 User Story 4 → 獨立測試 → 部署/展示
5. 每個故事增加價值而不破壞先前功能

### Single Developer Strategy

由於本專案為單一 HTML 檔案，建議：

1. 依序完成 Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5 → Phase 6
2. 每個 Phase 完成後進行 git commit
3. 每個 Checkpoint 進行視覺驗證

---

## Notes

- 本專案為單一 HTML 檔案架構，所有任務都在同一檔案中
- [P] 標記的任務表示可平行概念，但由於單檔案需注意程式碼區塊分離
- [Story] 標記將任務對應到特定使用者故事以便追蹤
- 每個使用者故事應可獨立完成和測試
- 每個任務或邏輯群組完成後進行 commit
- 在任何 Checkpoint 停止以獨立驗證故事
- 避免：模糊任務、跨故事依賴破壞獨立性

---

## Summary

| Metric | Value |
|--------|-------|
| **總任務數** | 38 |
| **Phase 1 (Setup)** | 4 任務 |
| **Phase 2 (Foundational)** | 7 任務 |
| **Phase 3 (US1 & US2 - MVP)** | 13 任務 |
| **Phase 4 (US3)** | 6 任務 |
| **Phase 5 (US4)** | 4 任務 |
| **Phase 6 (Polish)** | 4 任務 |
| **平行機會** | Setup、Stats、GUI 滑桿 |
| **建議 MVP 範圍** | Phase 1-3（US1 + US2）|
| **格式驗證** | ✅ 所有任務遵循 checklist 格式 |
