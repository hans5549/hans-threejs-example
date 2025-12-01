# Tasks: WebGL Video Kinect 深度影片點雲視覺化

**Input**: Design documents from `/specs/10-video-kinect/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Tests**: 無自動化測試需求 - 本功能為視覺展示範例，透過手動視覺測試驗證

**Organization**: 任務依據 User Story 組織，每個 Story 可獨立實作和測試

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: 可平行執行（不同檔案、無相依性）
- **[Story]**: 任務所屬的 User Story (US1, US2, US3)
- 描述包含確切檔案路徑

## Path Conventions

- **Target**: `examples/webgl-video-kinect/index.html` (單一 HTML 檔案)
- 所有程式碼內嵌於 index.html 中

---

## Phase 1: Setup (專案初始化)

**Purpose**: 建立基礎檔案結構和 HTML 框架

- [X] T001 建立專案目錄 `examples/webgl-video-kinect/`
- [X] T002 建立基礎 HTML 結構於 `examples/webgl-video-kinect/index.html`
  - DOCTYPE、head、body 標籤
  - meta viewport 設定
  - 基本 CSS 樣式（背景色、滿版、資訊標題）
- [X] T003 設定 ES Module importmap 於 `examples/webgl-video-kinect/index.html`
  - Three.js r160+ CDN 路徑
  - lil-gui addon 路徑

---

## Phase 2: Foundational (基礎架構)

**Purpose**: 核心元件初始化，為所有 User Story 建立基礎

**⚠️ CRITICAL**: 必須完成此階段，User Story 才能開始

- [X] T004 建立 `<video>` 元素於 `examples/webgl-video-kinect/index.html`
  - 設定 id="video"
  - 設定 loop, muted, crossOrigin, playsinline 屬性
  - 加入 webm 和 mp4 來源（使用 Three.js 官方 kinect 影片 URL）
  - 設定 display:none 隱藏
- [X] T005 建立 JavaScript module 區塊和匯入語句於 `examples/webgl-video-kinect/index.html`
  - import THREE
  - import GUI from lil-gui
- [X] T006 宣告全域變數於 `examples/webgl-video-kinect/index.html`
  - camera, scene, renderer
  - mouse 物件
  - center (Vector3)
  - windowHalfX, windowHalfY
- [X] T007 實作 WebGLRenderer 初始化於 `examples/webgl-video-kinect/index.html`
  - setPixelRatio
  - setSize
  - appendChild 到 body
- [X] T008 實作 PerspectiveCamera 初始化於 `examples/webgl-video-kinect/index.html`
  - FOV 50, near 1, far 10000
  - 初始位置 (0, 0, 500)
- [X] T009 實作 Scene 初始化於 `examples/webgl-video-kinect/index.html`
  - 建立 Scene 實例
  - 設定 center = Vector3(0, 0, -1000)

**Checkpoint**: 基礎渲染管線就緒

---

## Phase 3: User Story 1 - 深度影片點雲視覺化 (Priority: P1) 🎯 MVP

**Goal**: 將 Kinect 深度影片轉換為即時 3D 點雲顯示

**Independent Test**: 開啟頁面後，深度影片自動播放，可見 307,200 點的動態點雲

### Implementation for User Story 1

- [X] T010 [US1] 實作 VideoTexture 建立於 `examples/webgl-video-kinect/index.html`
  - 從 video 元素建立 VideoTexture
  - 設定 minFilter: NearestFilter
  - 設定 generateMipmaps: false
- [X] T011 [US1] 定義 Vertex Shader 程式碼於 `examples/webgl-video-kinect/index.html`
  - 宣告 uniforms (map, width, height, nearClipping, farClipping, pointSize, zOffset)
  - 定義 Kinect 常數 (XtoZ = 1.11146, YtoZ = 0.83359)
  - 實作深度取樣和 3D 座標轉換邏輯
  - 輸出 vUv, gl_PointSize, gl_Position
- [X] T012 [US1] 定義 Fragment Shader 程式碼於 `examples/webgl-video-kinect/index.html`
  - 接收 vUv varying
  - 從 map 取樣顏色
  - 輸出 gl_FragColor 含透明度 0.2
- [X] T013 [US1] 建立 uniforms 物件於 `examples/webgl-video-kinect/index.html`
  - map: VideoTexture
  - width: 640, height: 480
  - nearClipping: 850, farClipping: 4000
  - pointSize: 2, zOffset: 1000
- [X] T014 [US1] 建立 ShaderMaterial 於 `examples/webgl-video-kinect/index.html`
  - 傳入 uniforms, vertexShader, fragmentShader
  - 設定 blending: AdditiveBlending
  - 設定 depthTest: false, depthWrite: false
  - 設定 transparent: true
- [X] T015 [US1] 建立 BufferGeometry 點雲於 `examples/webgl-video-kinect/index.html`
  - 計算 640 * 480 = 307,200 個頂點位置
  - 初始化 Float32Array positions 陣列
  - 設定每個點的像素座標 (x, y, 0)
  - 設定 position attribute
- [X] T016 [US1] 建立 THREE.Points 物件於 `examples/webgl-video-kinect/index.html`
  - 結合 geometry 和 material
  - 加入 scene
- [X] T017 [US1] 實作影片播放控制於 `examples/webgl-video-kinect/index.html`
  - 呼叫 video.play()
- [X] T018 [US1] 實作基本 animate 迴圈於 `examples/webgl-video-kinect/index.html`
  - requestAnimationFrame(animate)
  - renderer.render(scene, camera)

**Checkpoint**: User Story 1 完成 - 點雲顯示並隨深度影片更新

---

## Phase 4: User Story 2 - 滑鼠互動相機控制 (Priority: P2)

**Goal**: 透過滑鼠移動改變相機視角，平滑探索 3D 場景

**Independent Test**: 移動滑鼠時，相機位置平滑跟隨，始終朝向點雲中心

### Implementation for User Story 2

- [X] T019 [US2] 實作 mousemove 事件處理於 `examples/webgl-video-kinect/index.html`
  - 計算滑鼠相對視窗中心的偏移
  - 乘以縮放因子 8
  - 更新 mouse.x 和 mouse.y
  - 註冊 document.addEventListener('mousemove', handler)
- [X] T020 [US2] 實作相機位置插值於 animate 函式中 `examples/webgl-video-kinect/index.html`
  - camera.position.x += (mouse.x - camera.position.x) * 0.05
  - camera.position.y += (-mouse.y - camera.position.y) * 0.05
- [X] T021 [US2] 實作相機 lookAt 於 animate 函式中 `examples/webgl-video-kinect/index.html`
  - camera.lookAt(center)

**Checkpoint**: User Story 2 完成 - 滑鼠控制相機流暢運作

---

## Phase 5: User Story 3 - GUI 參數即時調整 (Priority: P3)

**Goal**: 提供控制面板即時調整深度裁切、點大小、Z 偏移等參數

**Independent Test**: 調整 GUI 滑桿時，點雲效果立即改變

### Implementation for User Story 3

- [X] T022 [US3] 建立 lil-gui 實例於 `examples/webgl-video-kinect/index.html`
  - const gui = new GUI()
- [X] T023 [US3] 新增 nearClipping 控制項於 `examples/webgl-video-kinect/index.html`
  - gui.add(uniforms.nearClipping, 'value', 1, 10000, 1).name('Near Clipping')
- [X] T024 [P] [US3] 新增 farClipping 控制項於 `examples/webgl-video-kinect/index.html`
  - gui.add(uniforms.farClipping, 'value', 1, 10000, 1).name('Far Clipping')
- [X] T025 [P] [US3] 新增 pointSize 控制項於 `examples/webgl-video-kinect/index.html`
  - gui.add(uniforms.pointSize, 'value', 1, 10, 1).name('Point Size')
- [X] T026 [P] [US3] 新增 zOffset 控制項於 `examples/webgl-video-kinect/index.html`
  - gui.add(uniforms.zOffset, 'value', 0, 4000, 1).name('Z Offset')

**Checkpoint**: User Story 3 完成 - GUI 控制面板運作正常

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Edge cases 處理和最終優化

- [X] T027 實作 window resize 事件處理於 `examples/webgl-video-kinect/index.html`
  - 更新 windowHalfX, windowHalfY
  - 更新 camera.aspect
  - camera.updateProjectionMatrix()
  - renderer.setSize()
  - 註冊 window.addEventListener('resize', handler)
- [X] T028 加入 info 標題元素於 `examples/webgl-video-kinect/index.html`
  - 顯示 "three.js webgl - kinect depth video"
  - 包含 Three.js 官網連結
- [X] T029 執行 quickstart.md 驗證流程
  - 確認點雲正確顯示
  - 確認滑鼠控制流暢
  - 確認 GUI 參數即時生效
  - 確認效能達 30+ FPS

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1: Setup ─────────────────────────────┐
                                            ▼
Phase 2: Foundational ──────────────────────┤
                                            │
         ┌──────────────────────────────────┤
         ▼                                  │
Phase 3: User Story 1 (P1) 🎯 MVP          │
         │                                  │
         ▼                                  │
Phase 4: User Story 2 (P2) ◄────────────────┤
         │                                  │
         ▼                                  │
Phase 5: User Story 3 (P3) ◄────────────────┘
         │
         ▼
Phase 6: Polish
```

### User Story Dependencies

| User Story | 前置條件 | 可平行 | 備註 |
|------------|----------|--------|------|
| US1 (P1) | Phase 2 完成 | 否 | 核心功能，其他 Story 依賴此 |
| US2 (P2) | US1 完成 | 否 | 需要相機和動畫迴圈 |
| US3 (P3) | US1 完成 | 是 | 可與 US2 平行進行 |

### Within Each Phase

- Phase 3 (US1): T010 → T011/T012 (平行) → T013 → T014 → T015 → T016 → T017 → T018
- Phase 4 (US2): T019 → T020 → T021
- Phase 5 (US3): T022 → T023/T024/T025/T026 (平行)

### Parallel Opportunities

```bash
# Phase 3: Shader 定義可平行
Task T011 "定義 Vertex Shader"
Task T012 "定義 Fragment Shader"

# Phase 5: GUI 控制項可平行（僅 T023 必須先建立 GUI 實例）
Task T024 "新增 farClipping 控制項"
Task T025 "新增 pointSize 控制項"  
Task T026 "新增 zOffset 控制項"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. ✅ Complete Phase 1: Setup
2. ✅ Complete Phase 2: Foundational  
3. ✅ Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: 開啟頁面驗證點雲顯示
5. 若 MVP 滿足需求可先部署

### Incremental Delivery

1. Setup + Foundational → 基礎渲染管線就緒
2. User Story 1 → 測試點雲顯示 (MVP!)
3. User Story 2 → 測試滑鼠控制
4. User Story 3 → 測試 GUI 參數
5. Polish → 最終驗證

---

## Notes

- 所有程式碼都在單一 `index.html` 檔案中
- Shader 程式碼以字串常數形式內嵌
- 影片資源使用 Three.js 官方 CDN URL
- 無需額外安裝相依套件
- 開發時可使用任何本地 HTTP 伺服器（如 `python -m http.server`）測試
