# Tasks: WebGL Effects ASCII

**Input**: Design documents from `/specs/017-effects-ascii/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅, quickstart.md ✅

**Tests**: Not requested - manual browser testing only

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Project type**: Single HTML file example
- **Target file**: `examples/webgl-effects-ascii/index.html`

---

## Phase 1: Setup (Project Initialization)

**Purpose**: 建立專案目錄和基礎 HTML 結構

- [x] T001 Create directory structure at `examples/webgl-effects-ascii/`
- [x] T002 Create base HTML file with DOCTYPE, head, meta tags in `examples/webgl-effects-ascii/index.html`
- [x] T003 Add CSS reset styles (margin: 0, overflow: hidden, background: black) in `examples/webgl-effects-ascii/index.html`
- [x] T004 [P] Configure importmap for Three.js r170.0 CDN in `examples/webgl-effects-ascii/index.html`

**Checkpoint**: ✅ 基礎 HTML 結構就緒，可開始實作功能

---

## Phase 2: Foundational (Core Three.js Infrastructure)

**Purpose**: 建立 Three.js 核心基礎設施，所有 User Story 都依賴此階段

**⚠️ CRITICAL**: 必須完成此階段才能開始任何 User Story

- [x] T005 Import Three.js modules (THREE, AsciiEffect, TrackballControls) in `examples/webgl-effects-ascii/index.html`
- [x] T006 Declare global variables (camera, scene, renderer, effect, controls, sphere, start) in `examples/webgl-effects-ascii/index.html`
- [x] T007 Create init() function skeleton in `examples/webgl-effects-ascii/index.html`
- [x] T008 Create animate() function skeleton with requestAnimationFrame in `examples/webgl-effects-ascii/index.html`
- [x] T009 Add init() call at module load in `examples/webgl-effects-ascii/index.html`

**Checkpoint**: ✅ 基礎架構就緒 - 可開始 User Story 實作

---

## Phase 3: User Story 1 - 查看 ASCII 藝術渲染的 3D 場景 (Priority: P1) 🎯 MVP

**Goal**: 實現 ASCII 字元藝術風格渲染的 3D 場景，包含動態彈跳球體和平面

**Independent Test**: 開啟頁面後即可看到 ASCII 字元組成的 3D 場景，球體會自動上下彈跳並旋轉

### Implementation for User Story 1

- [x] T010 [US1] Create PerspectiveCamera (FOV 70, position y=150 z=500) in init() at `examples/webgl-effects-ascii/index.html`
- [x] T011 [US1] Create Scene with black background (0x000000) in init() at `examples/webgl-effects-ascii/index.html`
- [x] T012 [P] [US1] Create PointLight 1 (color 0xffffff, intensity 1.5, position 500,500,500) in init() at `examples/webgl-effects-ascii/index.html`
- [x] T013 [P] [US1] Create PointLight 2 (color 0xffffff, intensity 0.25, position -500,-500,-500) in init() at `examples/webgl-effects-ascii/index.html`
- [x] T014 [US1] Create Sphere mesh (SphereGeometry 200,20,10 + MeshPhongMaterial flatShading) in init() at `examples/webgl-effects-ascii/index.html`
- [x] T015 [US1] Create Plane mesh (PlaneGeometry 400,400 + MeshPhongMaterial, position y=-200, rotation x=-PI/2) in init() at `examples/webgl-effects-ascii/index.html`
- [x] T016 [US1] Create WebGLRenderer with window size in init() at `examples/webgl-effects-ascii/index.html`
- [x] T017 [US1] Create AsciiEffect with charset ' .:-+*=%@#' and options {invert: true} in init() at `examples/webgl-effects-ascii/index.html`
- [x] T018 [US1] Configure AsciiEffect domElement styles (color: white, backgroundColor: black) in init() at `examples/webgl-effects-ascii/index.html`
- [x] T019 [US1] Append effect.domElement to document.body in init() at `examples/webgl-effects-ascii/index.html`
- [x] T020 [US1] Implement sphere bounce animation (y = abs(sin(timer * 0.002)) * 150) in animate() at `examples/webgl-effects-ascii/index.html`
- [x] T021 [US1] Implement sphere rotation animation (x += timer * 0.0003, z += timer * 0.0002) in animate() at `examples/webgl-effects-ascii/index.html`
- [x] T022 [US1] Add effect.render(scene, camera) call in animate() at `examples/webgl-effects-ascii/index.html`

**Checkpoint**: ✅ User Story 1 完成 - ASCII 渲染場景應顯示彈跳旋轉的球體

---

## Phase 4: User Story 2 - 使用滑鼠控制相機視角 (Priority: P2)

**Goal**: 實現滑鼠控制相機視角（旋轉、縮放、平移）

**Independent Test**: 使用滑鼠左鍵拖曳旋轉、滾輪縮放、右鍵拖曳平移，相機視角應相應改變

### Implementation for User Story 2

- [x] T023 [US2] Create TrackballControls bound to camera and effect.domElement in init() at `examples/webgl-effects-ascii/index.html`
- [x] T024 [US2] Add controls.update() call in animate() at `examples/webgl-effects-ascii/index.html`

**Checkpoint**: ✅ User Story 2 完成 - 滑鼠可控制相機視角

---

## Phase 5: User Story 3 - 視窗大小自適應 (Priority: P3)

**Goal**: 實現視窗大小改變時自動調整渲染尺寸

**Independent Test**: 調整瀏覽器視窗大小，ASCII 渲染應自動調整以填滿新的視窗尺寸

### Implementation for User Story 3

- [x] T025 [US3] Create onWindowResize() function in `examples/webgl-effects-ascii/index.html`
- [x] T026 [US3] Update camera.aspect and camera.updateProjectionMatrix() in onWindowResize() at `examples/webgl-effects-ascii/index.html`
- [x] T027 [US3] Update renderer.setSize() in onWindowResize() at `examples/webgl-effects-ascii/index.html`
- [x] T028 [US3] Update effect.setSize() in onWindowResize() at `examples/webgl-effects-ascii/index.html`
- [x] T029 [US3] Add window resize event listener in init() at `examples/webgl-effects-ascii/index.html`

**Checkpoint**: ✅ User Story 3 完成 - 所有 User Story 功能就緒

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: 最終驗證和程式碼品質確認

- [x] T030 Verify HTML structure matches project conventions at `examples/webgl-effects-ascii/index.html`
- [ ] T031 Run quickstart.md validation - test all acceptance scenarios
- [ ] T032 Verify 30+ fps performance in browser DevTools

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1 (Setup)
    │
    ▼
Phase 2 (Foundational) ── BLOCKS ALL USER STORIES
    │
    ├─► Phase 3 (US1: ASCII Rendering) 🎯 MVP
    │       │
    │       ▼
    ├─► Phase 4 (US2: Camera Controls)
    │       │
    │       ▼
    └─► Phase 5 (US3: Responsive Resize)
            │
            ▼
        Phase 6 (Polish)
```

### User Story Dependencies

| User Story | Can Start After | Notes |
|------------|-----------------|-------|
| US1 (P1) | Phase 2 完成 | 核心功能，無其他依賴 |
| US2 (P2) | T019 完成 | 需要 effect.domElement 存在 |
| US3 (P3) | Phase 2 完成 | 需要 camera, renderer, effect 變數 |

### Within Each Phase

- 標記 [P] 的任務可平行執行
- T012 和 T013（兩個光源）可平行建立
- 場景物件建立後才能設定渲染器

### Parallel Opportunities

```bash
# Phase 1 可平行:
T002, T003, T004 (不同區塊，無相依)

# Phase 3 可平行:
T012, T013 (兩個光源，獨立物件)

# User Stories 之間:
US2 和 US3 可在 US1 完成後平行進行
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. 完成 Phase 1: Setup
2. 完成 Phase 2: Foundational (CRITICAL)
3. 完成 Phase 3: User Story 1
4. **STOP and VALIDATE**: 測試 ASCII 渲染是否正常
5. 如果 MVP 足夠，可先發布展示

### Incremental Delivery

1. Setup + Foundational → 基礎就緒
2. User Story 1 → ASCII 渲染 MVP 🎯
3. User Story 2 → 加入相機控制
4. User Story 3 → 加入響應式支援
5. 每個 Story 獨立可測試

### Single Developer Strategy

依優先順序執行：P1 → P2 → P3

---

## Notes

- 所有程式碼都在單一檔案 `examples/webgl-effects-ascii/index.html`
- [P] 任務 = 不同程式碼區塊，無相依性
- [Story] 標籤對應 spec.md 中的 User Story
- 每個 checkpoint 驗證該階段功能獨立運作
- 手動測試：開啟瀏覽器 → 檢視 Console → 驗證視覺效果
