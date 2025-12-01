# Tasks: WebGL Shader 動態視覺效果

**Input**: Design documents from `/specs/9-webgl-shader/`  
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: 專案初始化和基礎結構

- [ ] T001 Create project directory structure at examples/webgl-shader/
- [ ] T002 Create base HTML5 document with meta tags in examples/webgl-shader/index.html
- [ ] T003 [P] Configure Three.js import map (CDN unpkg r160) in examples/webgl-shader/index.html

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 核心基礎設施，所有使用者故事都依賴此階段

**⚠️ CRITICAL**: 此階段必須完成後才能開始使用者故事

- [ ] T004 [P] Implement inline CSS styles (body margin, overflow, info positioning) in examples/webgl-shader/index.html
- [ ] T005 [P] Create container div element (#container) in examples/webgl-shader/index.html
- [ ] T006 [P] Implement GLSL Vertex Shader in <script id="vertexShader"> in examples/webgl-shader/index.html
- [ ] T007 [P] Implement GLSL Fragment Shader (Monjori) in <script id="fragmentShader"> in examples/webgl-shader/index.html

**Checkpoint**: Foundation ready - 基礎 HTML 結構和 Shader 程式碼已就緒

---

## Phase 3: User Story 1 - 觀看動態 Shader 視覺效果 (Priority: P1) 🎯 MVP

**Goal**: 開啟網頁立即看到流暢運行的動態視覺效果

**Independent Test**: 開啟 index.html 確認畫面顯示流暢的動態色彩變化

### Implementation for User Story 1

- [ ] T008 [US1] Implement global variables declaration (camera, scene, renderer, uniforms) in examples/webgl-shader/index.html
- [ ] T009 [US1] Implement init() function - create OrthographicCamera(-1, 1, 1, -1, 0, 1) in examples/webgl-shader/index.html
- [ ] T010 [US1] Implement init() function - create Scene in examples/webgl-shader/index.html
- [ ] T011 [US1] Implement init() function - create PlaneGeometry(2, 2) in examples/webgl-shader/index.html
- [ ] T012 [US1] Implement init() function - create uniforms object { time: { value: 1.0 } } in examples/webgl-shader/index.html
- [ ] T013 [US1] Implement init() function - create ShaderMaterial with vertex/fragment shaders in examples/webgl-shader/index.html
- [ ] T014 [US1] Implement init() function - create Mesh and add to Scene in examples/webgl-shader/index.html
- [ ] T015 [US1] Implement init() function - create WebGLRenderer with pixelRatio and size in examples/webgl-shader/index.html
- [ ] T016 [US1] Implement init() function - append canvas to container and set animationLoop in examples/webgl-shader/index.html
- [ ] T017 [US1] Implement animate() function - update time uniform and render scene in examples/webgl-shader/index.html
- [ ] T018 [US1] Add init() call to start application in examples/webgl-shader/index.html

**Checkpoint**: User Story 1 完成 - 動態視覺效果應可正常顯示和運行

---

## Phase 4: User Story 2 - 響應式視窗調整 (Priority: P2)

**Goal**: 調整瀏覽器視窗大小時，視覺效果自動適應新尺寸

**Independent Test**: 改變瀏覽器視窗大小並觀察畫面是否正確調整

### Implementation for User Story 2

- [ ] T019 [US2] Implement onWindowResize() function - update renderer size in examples/webgl-shader/index.html
- [ ] T020 [US2] Add resize event listener in init() function in examples/webgl-shader/index.html

**Checkpoint**: User Story 2 完成 - 視窗調整後效果應正常填滿

---

## Phase 5: User Story 3 - 資訊展示與來源連結 (Priority: P3)

**Goal**: 頁面顯示相關資訊和來源連結

**Independent Test**: 檢查頁面上資訊文字和連結是否可點擊

### Implementation for User Story 3

- [ ] T021 [P] [US3] Create #info div with title text in examples/webgl-shader/index.html
- [ ] T022 [P] [US3] Add Three.js official website link (target="_blank") in examples/webgl-shader/index.html
- [ ] T023 [P] [US3] Add Monjori original work link (Pouët, target="_blank") in examples/webgl-shader/index.html
- [ ] T024 [US3] Style #info div for proper positioning and visibility in examples/webgl-shader/index.html

**Checkpoint**: User Story 3 完成 - 資訊和連結應正確顯示

---

## Phase 6: Polish & Validation

**Purpose**: 最終驗證和調整

- [ ] T025 Run visual verification - confirm dynamic effect displays immediately on page load
- [ ] T026 Run visual verification - confirm effect continues animating without stutter
- [ ] T027 Run visual verification - confirm window resize works correctly
- [ ] T028 Run visual verification - confirm links open in new tabs
- [ ] T029 [P] Browser test - Chrome latest version
- [ ] T030 [P] Browser test - Firefox latest version
- [ ] T031 Run quickstart.md validation

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1: Setup ──────────────┐
                             ▼
Phase 2: Foundational ───────┤
                             │
    ┌────────────────────────┼────────────────────────┐
    ▼                        ▼                        ▼
Phase 3: US1 (P1)      Phase 4: US2 (P2)       Phase 5: US3 (P3)
    │                        │                        │
    └────────────────────────┴────────────────────────┘
                             │
                             ▼
                    Phase 6: Polish
```

### User Story Dependencies

| User Story | 依賴 | 可平行 |
|------------|------|--------|
| US1 (P1) | Phase 2 完成 | 獨立 |
| US2 (P2) | US1 完成 (需要 renderer) | 依賴 US1 |
| US3 (P3) | Phase 2 完成 | 可與 US1 平行 |

### Within Each Phase

- Phase 1: T001 → T002 → T003
- Phase 2: T004, T005, T006, T007 皆可平行
- Phase 3: T008 → T009-T016 (順序執行) → T017 → T018
- Phase 4: T019 → T020
- Phase 5: T021, T022, T023 可平行 → T024
- Phase 6: 所有驗證任務可平行

### Parallel Opportunities

```bash
# Phase 2 - 可平行執行:
T004 (CSS styles)
T005 (container div)
T006 (Vertex Shader)
T007 (Fragment Shader)

# Phase 5 - 可平行執行:
T021 (info div)
T022 (Three.js link)
T023 (Monjori link)

# Phase 6 - 可平行執行:
T029 (Chrome test)
T030 (Firefox test)
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: 測試動態效果是否正常運行
5. 如果 MVP 滿足需求，可先部署

### Incremental Delivery

1. Setup + Foundational → 基礎就緒
2. Add User Story 1 → 獨立測試 → MVP 可用!
3. Add User Story 2 → 獨立測試 → 響應式支援
4. Add User Story 3 → 獨立測試 → 完整功能

---

## Task Summary

| 階段 | 任務數 | 預估時間 |
|------|--------|----------|
| Phase 1: Setup | 3 | 10 min |
| Phase 2: Foundational | 4 | 20 min |
| Phase 3: US1 (MVP) | 11 | 30 min |
| Phase 4: US2 | 2 | 10 min |
| Phase 5: US3 | 4 | 10 min |
| Phase 6: Polish | 7 | 15 min |
| **Total** | **31** | **~1.5 hr** |

---

## Notes

- [P] tasks = different files/sections, no dependencies
- [USx] label maps task to specific user story
- 此專案使用單一 HTML 檔案，所有任務都在同一檔案中
- 任務編號為連續執行順序
- 每個 checkpoint 後驗證該使用者故事是否獨立可用
- 避免: 模糊任務、跨故事依賴

