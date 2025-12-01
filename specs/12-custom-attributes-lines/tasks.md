# Tasks: WebGL Custom Attributes Lines

**Input**: Design documents from `/specs/12-custom-attributes-lines/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/component-interface.md ✅

**Tests**: Not requested - manual browser testing per plan.md
**Organization**: Tasks grouped by user story for independent implementation

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

Based on plan.md structure:
- **Output**: `examples/webgl-custom-attributes-lines/index.html`
- **Documentation**: `specs/12-custom-attributes-lines/`

---

## Phase 1: Setup (Project Initialization)

**Purpose**: Create directory structure and basic HTML framework

- [ ] T001 Create directory `examples/webgl-custom-attributes-lines/`
- [ ] T002 Create HTML boilerplate with DOCTYPE, head, and body structure in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T003 Add import map configuration for Three.js r174+ and addons in `examples/webgl-custom-attributes-lines/index.html`

**Checkpoint**: Basic HTML file exists with import map ready

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core rendering infrastructure that ALL user stories depend on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T004 Add CSS styles for full-viewport rendering in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T005 Add container div and info div elements in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T006 Add vertex shader script (id="vertexshader") with uniform/attribute/varying declarations in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T007 Add fragment shader script (id="fragmentshader") with color/opacity output in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T008 Implement WebGLRenderer initialization (antialias, pixelRatio, size) in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T009 Implement Scene and PerspectiveCamera setup (fov:30, z:400) in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T010 Implement uniforms object with amplitude, opacity, and color in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T011 Implement ShaderMaterial with AdditiveBlending, depthTest:false, transparent:true in `examples/webgl-custom-attributes-lines/index.html`

**Checkpoint**: Foundation ready - renderer, scene, camera, shaders, and material configured

---

## Phase 3: User Story 1 - 動態線條文字效果 (Priority: P1) 🎯 MVP

**Goal**: 使用者開啟網頁後看到「three.js」3D 文字以線條方式渲染，自動旋轉並產生動態位移效果

**Independent Test**: 開啟 `index.html`，應立即看到動態渲染的 3D 文字線條效果，線條波動且顏色漸變

### Implementation for User Story 1

- [ ] T012 [US1] Implement FontLoader to load Helvetiker Bold font from CDN in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T013 [US1] Implement init(font) function to create TextGeometry with specified parameters (size:50, depth:15) in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T014 [US1] Implement geometry.center() to center TextGeometry at origin in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T015 [US1] Implement displacement BufferAttribute setup (vec3 per vertex, random initial values) in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T016 [US1] Implement customColor BufferAttribute setup (vec3 per vertex, HSL-based colors) in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T017 [US1] Create THREE.Line with geometry and shaderMaterial, add to scene in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T018 [US1] Implement initWithFallback() function with BoxGeometry (100x100x100) for font load failure in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T019 [US1] Implement render() function with line.rotation.y animation (0.25 * time) in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T020 [US1] Implement amplitude uniform update with sin wave (0.5 * time) in render() in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T021 [US1] Implement color uniform HSL offset animation (0.0005 per frame) in render() in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T022 [US1] Implement displacement array random update with needsUpdate flag in render() in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T023 [US1] Implement animate() function with renderer.setAnimationLoop() in `examples/webgl-custom-attributes-lines/index.html`

**Checkpoint**: User Story 1 完成 - 3D 文字線條動畫可正常運作

---

## Phase 4: User Story 2 - 自適應視窗大小 (Priority: P2)

**Goal**: 使用者調整瀏覽器視窗大小時，3D 場景自動調整以適應新尺寸，保持正確比例

**Independent Test**: 調整瀏覽器視窗大小，場景應在 100ms 內正確重新渲染，無拉伸變形

### Implementation for User Story 2

- [ ] T024 [US2] Implement onWindowResize() function to update camera.aspect in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T025 [US2] Add camera.updateProjectionMatrix() call in onWindowResize() in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T026 [US2] Add renderer.setSize(window.innerWidth, window.innerHeight) in onWindowResize() in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T027 [US2] Add window.addEventListener('resize', onWindowResize) in initialization in `examples/webgl-custom-attributes-lines/index.html`

**Checkpoint**: User Story 2 完成 - 視窗調整時場景正確重新渲染

---

## Phase 5: User Story 3 - 效能監控顯示 (Priority: P3)

**Goal**: 使用者可以在畫面左上角看到即時 FPS 效能監控資訊

**Independent Test**: 開啟頁面後，左上角應顯示 Stats 效能監控面板，FPS 數值即時更新

### Implementation for User Story 3

- [ ] T028 [US3] Import Stats module from three/addons/libs/stats.module.js in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T029 [US3] Initialize Stats instance and append to document.body in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T030 [US3] Add stats.update() call in animate() function in `examples/webgl-custom-attributes-lines/index.html`

**Checkpoint**: User Story 3 完成 - Stats 面板顯示並即時更新

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final validation and documentation

- [ ] T031 [P] Add WebGL support detection with fallback message in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T032 [P] Add console.warn for font load failure in FontLoader error callback in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T033 [P] Update page title to "three.js webgl - custom attributes [lines]" in `examples/webgl-custom-attributes-lines/index.html`
- [ ] T034 Run quickstart.md validation (manual browser test on Chrome, Firefox, Safari, Edge)
- [ ] T035 Verify FPS ≥ 30 on target browsers
- [ ] T036 Update README.md to include new example link (if applicable)

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1: Setup           → No dependencies
Phase 2: Foundational    → Depends on Phase 1 (BLOCKS all user stories)
Phase 3: User Story 1    → Depends on Phase 2
Phase 4: User Story 2    → Depends on Phase 2 (can parallel with US1)
Phase 5: User Story 3    → Depends on Phase 2 (can parallel with US1, US2)
Phase 6: Polish          → Depends on Phase 3, 4, 5
```

### User Story Dependencies

| User Story | Depends On | Can Parallel With |
|------------|------------|-------------------|
| US1 (P1) | Foundational | - |
| US2 (P2) | Foundational | US1, US3 |
| US3 (P3) | Foundational | US1, US2 |

### Within Each Phase

- Phase 1: Sequential (T001 → T002 → T003)
- Phase 2: T004-T005 → T006-T007 (shaders) → T008-T009 (renderer/scene) → T010-T011 (uniforms/material)
- Phase 3: T012 → T013-T014 → T015-T016 (can parallel) → T017 → T018 → T019-T022 (render loop) → T023
- Phase 4: T024 → T025 → T026 → T027
- Phase 5: T028 → T029 → T030
- Phase 6: T031-T033 can parallel, T034-T036 sequential

### Parallel Opportunities

```bash
# Phase 2: Shaders can be written in parallel
Task T006: "Add vertex shader"
Task T007: "Add fragment shader"

# Phase 3: Buffer attributes can be set up in parallel
Task T015: "displacement BufferAttribute"
Task T016: "customColor BufferAttribute"

# User Stories can be implemented in parallel after Foundational
Task T024-T027 (US2): "Window resize handling"
Task T028-T030 (US3): "Stats integration"

# Phase 6: Polish tasks can run in parallel
Task T031: "WebGL detection"
Task T032: "Font load warning"
Task T033: "Page title"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. ✅ Complete Phase 1: Setup (T001-T003)
2. ✅ Complete Phase 2: Foundational (T004-T011)
3. ✅ Complete Phase 3: User Story 1 (T012-T023)
4. **STOP and VALIDATE**: Open in browser, verify 3D text animation
5. Deploy if ready - core demo functional

### Incremental Delivery

1. Setup + Foundational → HTML file with shaders and renderer ready
2. Add User Story 1 → Test independently → **MVP Complete!**
3. Add User Story 2 → Test resize → Enhanced user experience
4. Add User Story 3 → Test Stats panel → Developer tooling
5. Polish → Cross-browser validation → Production ready

### Single Developer Strategy (Recommended)

Since this is a single HTML file, work sequentially:

```
Day 1: Setup + Foundational (T001-T011)
Day 2: User Story 1 (T012-T023) → MVP Checkpoint
Day 3: User Story 2 + 3 (T024-T030) → Full Feature
Day 4: Polish (T031-T036) → Release Ready
```

---

## Notes

- All tasks target single file: `examples/webgl-custom-attributes-lines/index.html`
- No automated tests - manual browser testing per plan.md
- [P] tasks within same file should still be done sequentially
- [US#] labels track which user story each task belongs to
- Commit after each phase completion for clean history
- BoxGeometry fallback (T018) ensures graceful degradation
- Verify on multiple browsers before marking T034-T035 complete
