# Tasks: WebGL Geometry Terrain

**Input**: Design documents from `/specs/016-geometry-terrain/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Tests**: 未要求自動化測試，採用手動視覺測試 + 瀏覽器開發者工具驗證

**Organization**: Tasks are grouped by user story to enable independent implementation and testing.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

- **Project Type**: Single HTML (前端展示範例)
- **Target File**: `examples/webgl-geometry-terrain/index.html`

---

## Phase 1: Setup (Project Initialization)

**Purpose**: 建立專案基礎結構和 HTML 骨架

- [ ] T001 Create directory structure at examples/webgl-geometry-terrain/
- [ ] T002 Create HTML scaffold with DOCTYPE, head, meta tags in examples/webgl-geometry-terrain/index.html
- [ ] T003 Setup ES Module import map for Three.js CDN in examples/webgl-geometry-terrain/index.html
- [ ] T004 Add base CSS styles (body margin, overflow hidden, canvas display) in examples/webgl-geometry-terrain/index.html

**Checkpoint**: HTML 骨架完成，可在瀏覽器開啟顯示空白頁面

---

## Phase 2: Foundational (Core Infrastructure)

**Purpose**: 建立核心 Three.js 場景基礎設施，所有 User Story 都依賴這些元件

**⚠️ CRITICAL**: 所有 User Story 都需要 Scene, Camera, Renderer 才能運作

- [ ] T005 Create container element and fallback container in examples/webgl-geometry-terrain/index.html
- [ ] T006 Import Three.js core modules (THREE, Stats) in script section of examples/webgl-geometry-terrain/index.html
- [ ] T007 [P] Import ImprovedNoise from three/addons/math/ImprovedNoise.js in examples/webgl-geometry-terrain/index.html
- [ ] T008 [P] Import FirstPersonControls from three/addons/controls/FirstPersonControls.js in examples/webgl-geometry-terrain/index.html
- [ ] T009 Declare global variables (container, camera, scene, renderer, controls, clock, stats) in examples/webgl-geometry-terrain/index.html
- [ ] T010 Implement WebGL detection with try-catch in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T011 [P] Implement showFallback() function for WebGL degradation in examples/webgl-geometry-terrain/index.html

**Checkpoint**: 基礎設施就緒，可建立空白 Three.js 場景

---

## Phase 3: User Story 1 - 探索程序化生成的 3D 地形 (Priority: P1) 🎯 MVP

**Goal**: 使用者開啟頁面後能看到程序化生成的 3D 地形場景

**Independent Test**: 開啟頁面觀察是否顯示起伏的 3D 地形網格

### Implementation for User Story 1

- [ ] T012 [US1] Implement generateHeight(width, height) function using ImprovedNoise with 4 octaves in examples/webgl-geometry-terrain/index.html
- [ ] T013 [US1] Implement generateTexture(data, width, height) function with normal-based shading in examples/webgl-geometry-terrain/index.html
- [ ] T014 [US1] Create Scene instance in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T015 [US1] Create PerspectiveCamera with fov=60, near=1, far=10000 in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T016 [US1] Set camera initial position (100, 800, -800) and lookAt (-100, 810, -800) in examples/webgl-geometry-terrain/index.html
- [ ] T017 [US1] Call generateHeight(256, 256) to create height map data in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T018 [US1] Create PlaneGeometry(7500, 7500, 255, 255) for terrain in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T019 [US1] Rotate PlaneGeometry -PI/2 on X-axis to make it horizontal in examples/webgl-geometry-terrain/index.html
- [ ] T020 [US1] Apply height map data to geometry vertices (y = data[i] * 10) in examples/webgl-geometry-terrain/index.html
- [ ] T021 [US1] Call generateTexture() and create CanvasTexture with SRGBColorSpace in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T022 [US1] Create MeshBasicMaterial with terrain texture in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T023 [US1] Create terrain Mesh and add to scene in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T024 [US1] Create WebGLRenderer with antialias and set pixel ratio in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T025 [US1] Append renderer domElement to container in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T026 [US1] Implement basic animate() function with renderer.render() in examples/webgl-geometry-terrain/index.html
- [ ] T027 [US1] Call renderer.setAnimationLoop(animate) in init() function in examples/webgl-geometry-terrain/index.html

**Checkpoint**: 地形完整顯示，可看到程序化生成的高低起伏網格與明暗紋理

---

## Phase 4: User Story 2 - 使用第一人稱視角探索地形 (Priority: P2)

**Goal**: 使用者能使用第一人稱控制器自由移動和觀看地形

**Independent Test**: 使用滑鼠左鍵前進、右鍵後退，移動滑鼠旋轉視角

### Implementation for User Story 2

- [ ] T028 [US2] Create Clock instance for delta time calculation in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T029 [US2] Create FirstPersonControls with camera and renderer.domElement in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T030 [US2] Configure controls.movementSpeed = 150 in examples/webgl-geometry-terrain/index.html
- [ ] T031 [US2] Configure controls.lookSpeed = 0.1 in examples/webgl-geometry-terrain/index.html
- [ ] T032 [US2] Update animate() to call controls.update(clock.getDelta()) in examples/webgl-geometry-terrain/index.html

**Checkpoint**: 可使用滑鼠和鍵盤自由探索地形場景

---

## Phase 5: User Story 3 - 體驗視覺效果增強 (Priority: P3)

**Goal**: 場景具有指數霧效果和動態紋理，提升視覺沉浸感

**Independent Test**: 觀察遠近地形的霧效果差異和紋理明暗變化

### Implementation for User Story 3

- [ ] T033 [US3] Set scene.background = new THREE.Color(0xefd1b5) in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T034 [US3] Add scene.fog = new THREE.FogExp2(0xefd1b5, 0.0025) in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T035 [US3] Enhance generateTexture() with random noise (±5) for natural feel in examples/webgl-geometry-terrain/index.html
- [ ] T036 [US3] Scale texture canvas to 2x size for improved quality in generateTexture() in examples/webgl-geometry-terrain/index.html

**Checkpoint**: 霧效果使遠處地形融入背景，紋理具有自然的明暗和噪點變化

---

## Phase 6: User Story 4 - 響應式視窗調整 (Priority: P4)

**Goal**: 調整瀏覽器視窗大小時，場景正確適應新尺寸

**Independent Test**: 調整瀏覽器視窗大小，觀察場景是否正確更新且比例正確

### Implementation for User Story 4

- [ ] T037 [US4] Implement onWindowResize() function in examples/webgl-geometry-terrain/index.html
- [ ] T038 [US4] Add minimum window size check (200x200) in onWindowResize() in examples/webgl-geometry-terrain/index.html
- [ ] T039 [US4] Update camera.aspect and call camera.updateProjectionMatrix() in onWindowResize() in examples/webgl-geometry-terrain/index.html
- [ ] T040 [US4] Call renderer.setSize(window.innerWidth, window.innerHeight) in onWindowResize() in examples/webgl-geometry-terrain/index.html
- [ ] T041 [US4] Call controls.handleResize() in onWindowResize() in examples/webgl-geometry-terrain/index.html
- [ ] T042 [US4] Add window.addEventListener('resize', onWindowResize) in init() function in examples/webgl-geometry-terrain/index.html

**Checkpoint**: 視窗調整時場景自動適應，比例保持正確

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: 效能監控、最終驗證和文件更新

- [ ] T043 [P] Create Stats instance and append to document.body in init() function in examples/webgl-geometry-terrain/index.html
- [ ] T044 [P] Update animate() to call stats.update() in examples/webgl-geometry-terrain/index.html
- [ ] T045 [P] Add init() call at script end to start application in examples/webgl-geometry-terrain/index.html
- [ ] T046 Run quickstart.md validation - verify all controls and behaviors work as documented
- [ ] T047 Test WebGL fallback by simulating WebGL unavailable scenario
- [ ] T048 Verify 60 FPS performance using Stats panel

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1 (Setup)
    │
    ▼
Phase 2 (Foundational) ─── BLOCKS ALL USER STORIES
    │
    ├───────────────────────────────────────┐
    ▼                                       ▼
Phase 3 (US1: Terrain) ──────────────► Phase 4 (US2: Controls)
    │                                       │
    ▼                                       ▼
Phase 5 (US3: Visual Effects)         Phase 6 (US4: Responsive)
    │                                       │
    └───────────────┬───────────────────────┘
                    ▼
            Phase 7 (Polish)
```

### User Story Dependencies

- **User Story 1 (P1)**: 依賴 Phase 2 完成，無其他 User Story 依賴
- **User Story 2 (P2)**: 依賴 Phase 2 完成，需要 US1 的 Scene/Renderer 但可獨立測試
- **User Story 3 (P3)**: 依賴 Phase 2 完成，增強 US1 的視覺效果
- **User Story 4 (P4)**: 依賴 Phase 2 完成，需要 Camera/Renderer/Controls 存在

### Within Each User Story

- 按 Task ID 順序執行
- 標記 [P] 的任務可平行執行

### Parallel Opportunities

- **Phase 2**: T007 和 T008 可平行執行 (不同 import)，T011 可平行執行
- **Phase 3**: 大部分任務需依序執行 (相依性高)
- **Phase 7**: T043, T044, T045 可平行執行

---

## Implementation Strategy

### MVP Scope (Recommended)

**Minimum Viable Product = Phase 1 + Phase 2 + Phase 3 (User Story 1)**

完成 MVP 後，使用者可以看到程序化生成的 3D 地形，即使沒有控制功能也能展示核心價值。

### Incremental Delivery

1. **MVP**: T001-T027 (地形顯示)
2. **+Controls**: T028-T032 (互動功能)
3. **+Visual**: T033-T036 (視覺增強)
4. **+Responsive**: T037-T042 (響應式)
5. **+Polish**: T043-T048 (完善)

### Estimated Effort

| Phase | Task Count | Complexity | Est. Time |
|-------|------------|------------|-----------|
| Phase 1 | 4 | Low | 15 min |
| Phase 2 | 7 | Medium | 20 min |
| Phase 3 (US1) | 16 | High | 45 min |
| Phase 4 (US2) | 5 | Low | 15 min |
| Phase 5 (US3) | 4 | Low | 10 min |
| Phase 6 (US4) | 6 | Medium | 15 min |
| Phase 7 | 6 | Low | 10 min |
| **Total** | **48** | - | **~2.5 hrs** |

---

## Validation Checklist

### Per User Story

- [ ] US1: 開啟頁面後 3 秒內顯示地形
- [ ] US1: 地形具有明暗紋理
- [ ] US2: 滑鼠左鍵前進、右鍵後退
- [ ] US2: 滑鼠移動控制視角
- [ ] US3: 霧效果使遠處融入背景
- [ ] US3: 紋理具有自然噪點
- [ ] US4: 視窗調整後場景正確適應
- [ ] US4: 視窗小於 200x200 時停止渲染

### Edge Cases

- [ ] WebGL 不支援時顯示降級內容
- [ ] 背景標籤時動畫暫停 (瀏覽器自動處理)
- [ ] 快速點擊時移動基於時間差計算
