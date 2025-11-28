# Tasks: BufferGeometry DrawRange 互動式粒子網絡

**Input**: Design documents from `/specs/7-buffergeometry-drawrange/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Tests**: 無明確測試要求，僅進行手動瀏覽器測試驗證。

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

本功能使用單一 HTML 檔案結構：
- **主要檔案**: `examples/buffergeometry-drawrange/index.html`
- 所有 JavaScript 程式碼內嵌於 HTML 檔案中

---

## Phase 1: Setup

**Purpose**: 專案初始化與基礎結構

- [ ] T001 Create directory structure in examples/buffergeometry-drawrange/
- [ ] T002 Create base HTML5 document with meta tags in examples/buffergeometry-drawrange/index.html
- [ ] T003 [P] Add CSS styles for full-screen canvas and info overlay in examples/buffergeometry-drawrange/index.html
- [ ] T004 [P] Configure import map for Three.js r160 CDN dependencies in examples/buffergeometry-drawrange/index.html

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 核心 Three.js 場景基礎設施，必須在所有使用者故事之前完成

**⚠️ CRITICAL**: 無法開始任何使用者故事直到此階段完成

- [ ] T005 Setup global variables and constants (maxParticleCount, r, rHalf, etc.) in examples/buffergeometry-drawrange/index.html
- [ ] T006 Implement effectController object with default values in examples/buffergeometry-drawrange/index.html
- [ ] T007 Create PerspectiveCamera with FOV 45 and position z=1750 in examples/buffergeometry-drawrange/index.html
- [ ] T008 Create Scene and Group container in examples/buffergeometry-drawrange/index.html
- [ ] T009 Create WebGLRenderer with antialias and setAnimationLoop in examples/buffergeometry-drawrange/index.html
- [ ] T010 [P] Create BoxHelper boundary visualization (800x800x800) with additive blending in examples/buffergeometry-drawrange/index.html
- [ ] T011 [P] Setup Stats.js FPS counter in examples/buffergeometry-drawrange/index.html

**Checkpoint**: 基礎場景架構完成 - 可以看到空的場景和邊界框

---

## Phase 3: User Story 1 - 觀看 3D 粒子動畫 (Priority: P1) 🎯 MVP

**Goal**: 使用者可看到數百個白色粒子在立方體邊界內隨機移動

**Independent Test**: 開啟頁面後，觀察粒子是否持續在 3D 空間中移動並在邊界反彈

### Implementation for User Story 1

- [ ] T012 [US1] Create particlePositions Float32Array buffer (maxParticleCount × 3) in examples/buffergeometry-drawrange/index.html
- [ ] T013 [US1] Initialize random particle positions within boundary (-rHalf to rHalf) in examples/buffergeometry-drawrange/index.html
- [ ] T014 [US1] Create particlesData array with velocity vectors and numConnections in examples/buffergeometry-drawrange/index.html
- [ ] T015 [US1] Create particles BufferGeometry with DynamicDrawUsage position attribute in examples/buffergeometry-drawrange/index.html
- [ ] T016 [US1] Create PointsMaterial with white color, size 3, AdditiveBlending, sizeAttenuation false in examples/buffergeometry-drawrange/index.html
- [ ] T017 [US1] Create pointCloud (THREE.Points) and add to group in examples/buffergeometry-drawrange/index.html
- [ ] T018 [US1] Set particles.setDrawRange(0, particleCount) for initial render count in examples/buffergeometry-drawrange/index.html
- [ ] T019 [US1] Implement animate() function with particle position updates in examples/buffergeometry-drawrange/index.html
- [ ] T020 [US1] Implement boundary collision detection and velocity reversal in examples/buffergeometry-drawrange/index.html
- [ ] T021 [US1] Set particles.geometry.attributes.position.needsUpdate = true each frame in examples/buffergeometry-drawrange/index.html
- [ ] T022 [US1] Implement render() function with group rotation based on time in examples/buffergeometry-drawrange/index.html

**Checkpoint**: User Story 1 完成 - 粒子應持續移動、反彈，場景緩慢旋轉

---

## Phase 4: User Story 2 - 動態粒子連線效果 (Priority: P1)

**Goal**: 距離接近的粒子自動產生連線，透明度隨距離變化

**Independent Test**: 觀察接近的粒子之間是否出現連線，線條透明度隨距離變化

### Implementation for User Story 2

- [ ] T023 [US2] Create positions Float32Array for line segments (maxParticleCount² × 3) in examples/buffergeometry-drawrange/index.html
- [ ] T024 [P] [US2] Create colors Float32Array for line segment colors (maxParticleCount² × 3) in examples/buffergeometry-drawrange/index.html
- [ ] T025 [US2] Create line geometry BufferGeometry with DynamicDrawUsage position and color attributes in examples/buffergeometry-drawrange/index.html
- [ ] T026 [US2] Create LineBasicMaterial with vertexColors, AdditiveBlending, transparent in examples/buffergeometry-drawrange/index.html
- [ ] T027 [US2] Create linesMesh (THREE.LineSegments) and add to group in examples/buffergeometry-drawrange/index.html
- [ ] T028 [US2] Set line geometry.computeBoundingSphere() and setDrawRange(0, 0) in examples/buffergeometry-drawrange/index.html
- [ ] T029 [US2] Reset all particleData.numConnections to 0 at start of each animate() frame in examples/buffergeometry-drawrange/index.html
- [ ] T030 [US2] Implement O(n²) distance calculation loop between particles in animate() in examples/buffergeometry-drawrange/index.html
- [ ] T031 [US2] Calculate alpha transparency based on distance (alpha = 1.0 - dist/minDistance) in examples/buffergeometry-drawrange/index.html
- [ ] T032 [US2] Write line segment vertex positions for connected particles in examples/buffergeometry-drawrange/index.html
- [ ] T033 [US2] Write line segment colors with calculated alpha values in examples/buffergeometry-drawrange/index.html
- [ ] T034 [US2] Increment numConnections counters for both particles in connection in examples/buffergeometry-drawrange/index.html
- [ ] T035 [US2] Check limitConnections and maxConnections before creating connection in examples/buffergeometry-drawrange/index.html
- [ ] T036 [US2] Update linesMesh.geometry.setDrawRange(0, numConnected × 2) each frame in examples/buffergeometry-drawrange/index.html
- [ ] T037 [US2] Set linesMesh.geometry.attributes.position.needsUpdate = true each frame in examples/buffergeometry-drawrange/index.html
- [ ] T038 [US2] Set linesMesh.geometry.attributes.color.needsUpdate = true each frame in examples/buffergeometry-drawrange/index.html

**Checkpoint**: User Story 2 完成 - 連線應根據距離動態顯示/消失，透明度隨距離變化

---

## Phase 5: User Story 3 - GUI 控制面板互動 (Priority: P2)

**Goal**: 使用者可透過控制面板調整視覺化參數

**Independent Test**: 調整各項參數並觀察場景即時變化

### Implementation for User Story 3

- [ ] T039 [US3] Import GUI from three/addons/libs/lil-gui.module.min.js in examples/buffergeometry-drawrange/index.html
- [ ] T040 [US3] Implement initGUI() function creating new GUI instance in examples/buffergeometry-drawrange/index.html
- [ ] T041 [US3] Add showDots checkbox with onChange toggling pointCloud.visible in examples/buffergeometry-drawrange/index.html
- [ ] T042 [P] [US3] Add showLines checkbox with onChange toggling linesMesh.visible in examples/buffergeometry-drawrange/index.html
- [ ] T043 [P] [US3] Add minDistance slider (10-300) reading from effectController in animate() in examples/buffergeometry-drawrange/index.html
- [ ] T044 [P] [US3] Add limitConnections checkbox reading from effectController in animate() in examples/buffergeometry-drawrange/index.html
- [ ] T045 [P] [US3] Add maxConnections slider (0-30, step 1) reading from effectController in animate() in examples/buffergeometry-drawrange/index.html
- [ ] T046 [US3] Add particleCount slider (0-1000, step 1) with onChange updating particleCount and particles.setDrawRange in examples/buffergeometry-drawrange/index.html
- [ ] T047 [US3] Call initGUI() at beginning of init() function in examples/buffergeometry-drawrange/index.html

**Checkpoint**: User Story 3 完成 - 所有 GUI 控制項應即時影響場景

---

## Phase 6: User Story 4 - 3D 攝影機軌道控制 (Priority: P2)

**Goal**: 使用者可透過滑鼠旋轉、縮放和平移視角

**Independent Test**: 滑鼠左鍵拖曳旋轉、滾輪縮放、右鍵平移

### Implementation for User Story 4

- [ ] T048 [US4] Import OrbitControls from three/addons/controls/OrbitControls.js in examples/buffergeometry-drawrange/index.html
- [ ] T049 [US4] Create OrbitControls instance attached to camera and container in examples/buffergeometry-drawrange/index.html
- [ ] T050 [US4] Set controls.minDistance = 1000 for zoom limit in examples/buffergeometry-drawrange/index.html
- [ ] T051 [US4] Set controls.maxDistance = 3000 for zoom limit in examples/buffergeometry-drawrange/index.html

**Checkpoint**: User Story 4 完成 - 攝影機控制應流暢運作

---

## Phase 7: User Story 5 - 視窗大小自適應 (Priority: P3)

**Goal**: 瀏覽器視窗大小變化時自動調整場景

**Independent Test**: 調整視窗大小，確認場景自動適應且長寬比正確

### Implementation for User Story 5

- [ ] T052 [US5] Implement onWindowResize() function in examples/buffergeometry-drawrange/index.html
- [ ] T053 [US5] Update camera.aspect in onWindowResize() in examples/buffergeometry-drawrange/index.html
- [ ] T054 [US5] Call camera.updateProjectionMatrix() in onWindowResize() in examples/buffergeometry-drawrange/index.html
- [ ] T055 [US5] Call renderer.setSize(window.innerWidth, window.innerHeight) in onWindowResize() in examples/buffergeometry-drawrange/index.html
- [ ] T056 [US5] Add window.addEventListener('resize', onWindowResize) in init() in examples/buffergeometry-drawrange/index.html

**Checkpoint**: User Story 5 完成 - 視窗調整應自動適應

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: 最終驗證與優化

- [ ] T057 [P] Verify all FR requirements against spec.md checklist
- [ ] T058 [P] Test with 200 particles for 60 FPS performance target
- [ ] T059 [P] Test with 500 particles for 30+ FPS performance target
- [ ] T060 [P] Test edge case: particleCount = 0
- [ ] T061 [P] Test edge case: minDistance = 300 (maximum)
- [ ] T062 [P] Test edge case: maxConnections = 1
- [ ] T063 Run quickstart.md validation with local server
- [ ] T064 Update README.md to include new example link (if applicable)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-7)**: All depend on Foundational phase completion
  - User Story 1 (P1): MVP - complete first
  - User Story 2 (P1): Builds on US1 particle system
  - User Stories 3, 4, 5 (P2/P3): Can proceed after US1+US2

### User Story Dependencies

- **User Story 1 (P1)**: Depends on Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P1)**: Depends on User Story 1 (requires particle system)
- **User Story 3 (P2)**: Depends on User Story 1 + 2 (controls particle/line visibility)
- **User Story 4 (P2)**: Depends on Foundational only - can run parallel to US1
- **User Story 5 (P3)**: Depends on Foundational only - can run parallel to US1

### Within Each User Story

- Core implementation before dependent features
- Buffer creation before geometry setup
- Geometry before material
- All components before animate() integration

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel (T003, T004)
- Foundational tasks T010, T011 can run in parallel
- US2 buffer creation T023, T024 can run in parallel
- US3 GUI controls T042-T045 can run in parallel
- All Polish tasks marked [P] can run in parallel

---

## Parallel Example: User Story 2

```bash
# Launch buffer creation in parallel:
Task: "Create positions Float32Array for line segments"
Task: "Create colors Float32Array for line segment colors"

# After buffers complete, proceed with geometry setup
```

---

## Implementation Strategy

### MVP First (User Stories 1 + 2)

1. Complete Phase 1: Setup (T001-T004)
2. Complete Phase 2: Foundational (T005-T011)
3. Complete Phase 3: User Story 1 (T012-T022)
4. **CHECKPOINT**: 驗證粒子移動和旋轉
5. Complete Phase 4: User Story 2 (T023-T038)
6. **MVP COMPLETE**: 粒子網絡視覺化可用

### Incremental Delivery

1. Setup + Foundational → 場景基礎就緒
2. User Story 1 → 粒子動畫可用
3. User Story 2 → 連線效果可用 (MVP!)
4. User Story 3 → GUI 控制可用
5. User Story 4 → 攝影機控制可用
6. User Story 5 → 響應式佈局可用
7. Polish → 效能驗證和邊界測試

---

## Notes

- [P] tasks = different code sections, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently verifiable via browser testing
- Commit after each phase completion
- All code in single file: examples/buffergeometry-drawrange/index.html
- Total tasks: 64
- Tasks per user story: US1=11, US2=16, US3=9, US4=4, US5=5
