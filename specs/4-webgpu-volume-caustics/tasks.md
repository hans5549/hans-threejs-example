# Tasks: WebGPU Volume Caustics

**Input**: Design documents from `/specs/4-webgpu-volume-caustics/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Tests**: 不包含自動化測試任務（展示範例專案，手動瀏覽器測試為主）

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

- **Project Type**: Single HTML file
- **Target Path**: `examples/webgpu-volume-caustics/index.html`

---

## Phase 1: Setup (Project Initialization)

**Purpose**: 建立專案基本結構和 WebGPU 偵測邏輯

- [ ] T001 Create directory `examples/webgpu-volume-caustics/`
- [ ] T002 Create base HTML structure with meta tags and title in `examples/webgpu-volume-caustics/index.html`
- [ ] T003 Add ES Modules import map for Three.js WebGPU dependencies in `examples/webgpu-volume-caustics/index.html`
- [ ] T004 Implement WebGPU support detection logic (`navigator.gpu` check) in `examples/webgpu-volume-caustics/index.html`
- [ ] T005 Create fallback UI for unsupported browsers (static image + message) in `examples/webgpu-volume-caustics/index.html`
- [ ] T006 Add basic CSS styles for info display and fallback UI in `examples/webgpu-volume-caustics/index.html`

**Checkpoint**: 頁面可在瀏覽器開啟，支援 WebGPU 時準備初始化，不支援時顯示 fallback

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 建立 Three.js 場景核心基礎設施，所有 user stories 都依賴此階段

**⚠️ CRITICAL**: 必須完成此階段才能開始任何 user story 實作

- [ ] T007 Initialize async `init()` function structure in `examples/webgpu-volume-caustics/index.html`
- [ ] T008 Create THREE.WebGPURenderer with antialias and shadow map enabled in `examples/webgpu-volume-caustics/index.html`
- [ ] T009 Create THREE.PerspectiveCamera with FOV 25, position (-0.7, 0.2, 0.2) in `examples/webgpu-volume-caustics/index.html`
- [ ] T010 Create THREE.Scene container in `examples/webgpu-volume-caustics/index.html`
- [ ] T011 [P] Configure THREE.SpotLight with HDR shadow (HalfFloatType) in `examples/webgpu-volume-caustics/index.html`
- [ ] T012 [P] Create ground PlaneGeometry (2x2) with MeshStandardMaterial, receiveShadow=true in `examples/webgpu-volume-caustics/index.html`
- [ ] T013 Setup render size to window dimensions in `examples/webgpu-volume-caustics/index.html`
- [ ] T014 Append renderer.domElement to document.body in `examples/webgpu-volume-caustics/index.html`

**Checkpoint**: Foundation ready - 可見聚光燈照射的地面場景

---

## Phase 3: User Story 1 - 觀看即時體積焦散效果 (Priority: P1) 🎯 MVP

**Goal**: 使用者可觀看透明鴨子模型產生的即時焦散光影效果，地面顯示動態彩虹光斑

**Independent Test**: 頁面載入後可看到透明鴨子，地面上有彩虹色焦散光斑

### Implementation for User Story 1

- [ ] T015 [US1] Configure GLTFLoader with DRACOLoader for model loading in `examples/webgpu-volume-caustics/index.html`
- [ ] T016 [US1] Load duck.glb model from Three.js CDN path in `examples/webgpu-volume-caustics/index.html`
- [ ] T017 [US1] Create MeshPhysicalNodeMaterial with transmission=1, ior=1.5, thickness=0.25 in `examples/webgpu-volume-caustics/index.html`
- [ ] T018 [US1] Apply transparent glass material to duck mesh with DoubleSide rendering in `examples/webgpu-volume-caustics/index.html`
- [ ] T019 [US1] Load caustic texture (Caustic_Free.jpg) with RepeatWrapping in `examples/webgpu-volume-caustics/index.html`
- [ ] T020 [US1] Implement TSL `causticEffect` Fn() with refract calculation in `examples/webgpu-volume-caustics/index.html`
- [ ] T021 [US1] Implement chromatic aberration RGB offset in caustic shader in `examples/webgpu-volume-caustics/index.html`
- [ ] T022 [US1] Assign causticEffect to duck.material.castShadowNode in `examples/webgpu-volume-caustics/index.html`
- [ ] T023 [US1] Implement TSL emissiveNode for backside scattering effect in `examples/webgpu-volume-caustics/index.html`
- [ ] T024 [US1] Enable duck.castShadow = true in `examples/webgpu-volume-caustics/index.html`
- [ ] T025 [US1] Implement animate() function with duck rotation (rotation.y -= 0.01) in `examples/webgpu-volume-caustics/index.html`
- [ ] T026 [US1] Setup renderer.setAnimationLoop(animate) in `examples/webgpu-volume-caustics/index.html`

**Checkpoint**: User Story 1 完成 - 透明鴨子旋轉，地面顯示彩虹焦散效果 ✅

---

## Phase 4: User Story 2 - 互動式視角控制 (Priority: P1)

**Goal**: 使用者可透過滑鼠/觸控操作旋轉和縮放視角

**Independent Test**: 滑鼠拖曳可旋轉視角，滾輪可縮放

### Implementation for User Story 2

- [ ] T027 [US2] Import OrbitControls from three/addons in `examples/webgpu-volume-caustics/index.html`
- [ ] T028 [US2] Create OrbitControls instance with camera and renderer.domElement in `examples/webgpu-volume-caustics/index.html`
- [ ] T029 [US2] Configure controls.target to (0, 0.02, -0.05) in `examples/webgpu-volume-caustics/index.html`
- [ ] T030 [US2] Set controls.maxDistance = 1 to limit zoom out in `examples/webgpu-volume-caustics/index.html`
- [ ] T031 [US2] Add controls.update() to animate loop in `examples/webgpu-volume-caustics/index.html`

**Checkpoint**: User Story 2 完成 - 相機可互動旋轉和縮放 ✅

---

## Phase 5: User Story 3 - 體積光效果展示 (Priority: P2)

**Goal**: 場景中可見霧氣中的體積光束效果

**Independent Test**: 可見聚光燈光束穿透煙霧效果

### Implementation for User Story 3

- [ ] T032 [US3] Import ImprovedNoise from three/addons/math in `examples/webgpu-volume-caustics/index.html`
- [ ] T033 [US3] Generate 64x64x64 3D noise texture using ImprovedNoise in `examples/webgpu-volume-caustics/index.html`
- [ ] T034 [US3] Create THREE.Data3DTexture with RedFormat and LinearFilter in `examples/webgpu-volume-caustics/index.html`
- [ ] T035 [US3] Import VolumeNodeMaterial from three/webgpu in `examples/webgpu-volume-caustics/index.html`
- [ ] T036 [US3] Create VolumeNodeMaterial with spotlight reference in `examples/webgpu-volume-caustics/index.html`
- [ ] T037 [US3] Implement TSL scatteringNode with 3D noise sampling in `examples/webgpu-volume-caustics/index.html`
- [ ] T038 [US3] Create volumetric BoxGeometry mesh (1.5 x 0.5 x 1.5) in `examples/webgpu-volume-caustics/index.html`
- [ ] T039 [US3] Configure LAYER_VOLUMETRIC_LIGHTING = 10 for layer separation in `examples/webgpu-volume-caustics/index.html`
- [ ] T040 [US3] Set volumetricMesh.layers.enable(LAYER_VOLUMETRIC_LIGHTING) in `examples/webgpu-volume-caustics/index.html`

**Checkpoint**: User Story 3 完成 - 體積光效果可見 ✅

---

## Phase 6: User Story 3 - 後處理整合 (Priority: P2 續)

**Goal**: 完成後處理管線，加入 Bloom 光暈效果

**Independent Test**: 體積光具有柔和的 Bloom 光暈

### Implementation for User Story 3 (Post Processing)

- [ ] T041 [US3] Import PostProcessing class from three/webgpu in `examples/webgpu-volume-caustics/index.html`
- [ ] T042 [US3] Import pass and bloom from three/tsl and three/addons in `examples/webgpu-volume-caustics/index.html`
- [ ] T043 [US3] Create PostProcessing instance with renderer in `examples/webgpu-volume-caustics/index.html`
- [ ] T044 [US3] Create scenePass with pass(scene, camera) in `examples/webgpu-volume-caustics/index.html`
- [ ] T045 [US3] Get sceneDepth texture node from scenePass in `examples/webgpu-volume-caustics/index.html`
- [ ] T046 [US3] Set volumetricMaterial.depthNode to sceneDepth for occlusion in `examples/webgpu-volume-caustics/index.html`
- [ ] T047 [US3] Create volumetricPass with layer filtering (setLayers) in `examples/webgpu-volume-caustics/index.html`
- [ ] T048 [US3] Set volumetricPass.setResolutionScale(0.5) for performance in `examples/webgpu-volume-caustics/index.html`
- [ ] T049 [US3] Create bloomPass with bloom(volumetricPass, 1, 1, 0) in `examples/webgpu-volume-caustics/index.html`
- [ ] T050 [US3] Composite final output: scenePass + bloomPass * intensity in `examples/webgpu-volume-caustics/index.html`
- [ ] T051 [US3] Update animate() to use postProcessing.render() instead of renderer.render() in `examples/webgpu-volume-caustics/index.html`

**Checkpoint**: User Story 3 完全完成 - 體積光 + Bloom 效果可見 ✅

---

## Phase 7: User Story 4 - 響應式視窗調整 (Priority: P3)

**Goal**: 視窗大小改變時畫面自動適應

**Independent Test**: 調整視窗大小後畫面正確適應

### Implementation for User Story 4

- [ ] T052 [US4] Implement onWindowResize() function in `examples/webgpu-volume-caustics/index.html`
- [ ] T053 [US4] Update camera.aspect and camera.updateProjectionMatrix() in resize handler in `examples/webgpu-volume-caustics/index.html`
- [ ] T054 [US4] Update renderer.setSize(innerWidth, innerHeight) in resize handler in `examples/webgpu-volume-caustics/index.html`
- [ ] T055 [US4] Add window.addEventListener('resize', onWindowResize) in `examples/webgpu-volume-caustics/index.html`

**Checkpoint**: User Story 4 完成 - 響應式視窗調整 ✅

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: 最終調整和品質改善

- [ ] T056 [P] Add info div with title "Volumetric Caustics" and description in `examples/webgpu-volume-caustics/index.html`
- [ ] T057 [P] Add loading indicator during asset loading in `examples/webgpu-volume-caustics/index.html`
- [ ] T058 Add error handling for model load failure in `examples/webgpu-volume-caustics/index.html`
- [ ] T059 Add error handling for texture load failure in `examples/webgpu-volume-caustics/index.html`
- [ ] T060 Final code cleanup and remove unused variables in `examples/webgpu-volume-caustics/index.html`
- [ ] T061 Validate quickstart.md instructions work correctly
- [ ] T062 Manual browser testing in Chrome 113+ and Edge 113+
- [ ] T063 Verify fallback UI works in Safari/Firefox (no WebGPU)

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1 (Setup)
    ↓
Phase 2 (Foundational) ← BLOCKS all user stories
    ↓
┌───────────────────────────────────────────────┐
│ Phase 3 (US1: 焦散效果)     [P1] 🎯 MVP       │
│ Phase 4 (US2: 視角控制)     [P1]              │
│   ↑ Can run in parallel after Phase 2         │
└───────────────────────────────────────────────┘
    ↓
Phase 5-6 (US3: 體積光 + 後處理) [P2]
    ↓
Phase 7 (US4: 響應式)            [P3]
    ↓
Phase 8 (Polish)
```

### User Story Dependencies

| User Story | 優先序 | 前置條件 | 可並行 |
|------------|--------|----------|--------|
| US1: 焦散效果 | P1 | Phase 2 完成 | ✅ (與 US2) |
| US2: 視角控制 | P1 | Phase 2 完成 | ✅ (與 US1) |
| US3: 體積光 | P2 | Phase 2 完成, 建議 US1 完成 | ❌ (需 post-processing 整合) |
| US4: 響應式 | P3 | Phase 2 完成 | ✅ (獨立) |

### Within Each User Story

- 資源載入 (textures, models) 優先
- TSL 著色器實作次之
- 材質套用和整合最後
- 每個 story 完成後應可獨立測試

### Parallel Opportunities per Phase

**Phase 1 (Setup)**:
```
T001 → T002 → T003 → T004 → T005, T006 [P]
```

**Phase 2 (Foundational)**:
```
T007 → T008 → T009, T010 [P] → T011, T012 [P] → T013 → T014
```

**Phase 3 (US1)**:
```
T015 → T016 → T017 → T018
T019 (可並行)
T020 → T021 → T022 → T023
T024 → T025 → T026
```

**Phase 4 (US2)**: 
```
T027 → T028 → T029 → T030 → T031
```

---

## Implementation Strategy

### MVP 範圍 (Minimum Viable Product)

優先完成 Phase 1-4，達成核心焦散效果 + 視角控制：

1. ✅ Setup (Phase 1)
2. ✅ Foundational (Phase 2)  
3. ✅ US1: 焦散效果 (Phase 3) - **核心功能**
4. ✅ US2: 視角控制 (Phase 4) - **基本互動**

### 增量交付順序

1. **MVP**: Phase 1-4 (焦散 + 控制)
2. **Enhanced**: Phase 5-6 (體積光)
3. **Complete**: Phase 7-8 (響應式 + Polish)

---

## Task Count Summary

| Phase | Tasks | User Story |
|-------|-------|------------|
| Phase 1: Setup | 6 | - |
| Phase 2: Foundational | 8 | - |
| Phase 3: US1 | 12 | US1 - 焦散效果 |
| Phase 4: US2 | 5 | US2 - 視角控制 |
| Phase 5: US3 Part 1 | 9 | US3 - 體積光 |
| Phase 6: US3 Part 2 | 11 | US3 - 後處理 |
| Phase 7: US4 | 4 | US4 - 響應式 |
| Phase 8: Polish | 8 | - |
| **Total** | **63** | |

### Parallel Opportunities Identified

- Phase 1: 2 tasks [P]
- Phase 2: 4 tasks [P]
- Phase 8: 2 tasks [P]
- Cross-story: US1 和 US2 可同時進行
