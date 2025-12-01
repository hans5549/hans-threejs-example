# Tasks: Particle Earth Sphere

**Input**: Design documents from `/specs/14-particle-earth/`  
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: 專案初始化和基礎結構

- [ ] T001 建立專案目錄結構 `examples/particle-earth-sphere/`
- [ ] T002 [P] 建立資源目錄 `examples/particle-earth-sphere/assets/`
- [ ] T003 [P] 下載 NASA Blue Marble 地球紋理至 `examples/particle-earth-sphere/assets/earth-map.jpg`
- [ ] T004 [P] 準備 WebGL 降級用靜態圖片 `examples/particle-earth-sphere/assets/earth-fallback.jpg`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 核心基礎設施，所有 User Story 開始前必須完成

**⚠️ CRITICAL**: 此階段完成後才能開始 User Story 實作

- [ ] T005 建立 HTML 基礎結構含 meta 標籤和 CSS 樣式於 `examples/particle-earth-sphere/index.html`
- [ ] T006 設定 ES Module importmap 引入 Three.js 和 lil-gui CDN
- [ ] T007 實作 WebGL 可用性檢測和降級顯示邏輯
- [ ] T008 建立 Three.js Scene, PerspectiveCamera, WebGLRenderer 初始化程式碼
- [ ] T009 實作 TextureLoader 載入地球紋理並處理載入完成回呼
- [ ] T010 實作視窗 resize 事件處理器更新攝影機和渲染器尺寸

**Checkpoint**: 基礎架構就緒 - 可以開始平行實作各 User Story

---

## Phase 3: User Story 1 - 檢視粒子地球 (Priority: P1) 🎯 MVP

**Goal**: 使用者載入網頁後可見由粒子組成的地球球體，且自動旋轉

**Independent Test**: 開啟網頁後立即可見粒子組成的地球球體，球體正在自動旋轉

### Implementation for User Story 1

- [ ] T011 [US1] 實作 `generateFibonacciSphere(numParticles, radius)` 函式產生 50,000 個粒子的球面分布座標
- [ ] T012 [US1] 建立 BufferGeometry 並設定 position attribute 使用 Float32Array
- [ ] T013 [US1] 編寫 Vertex Shader 含 Y 軸旋轉、UV 座標計算、紋理取樣
- [ ] T014 [US1] 編寫 Fragment Shader 使用 vColor 和 opacity
- [ ] T015 [US1] 建立 ShaderMaterial 設定 uniforms、blending 模式和透明度
- [ ] T016 [US1] 建立 Points mesh 並加入 Scene
- [ ] T017 [US1] 實作 animate() 動畫迴圈更新 time uniform 並呼叫 renderer.render()
- [ ] T018 [US1] 驗證粒子地球顯示正確，紋理顏色對應地球大陸分布

**Checkpoint**: User Story 1 完成，可獨立展示自動旋轉的粒子地球

---

## Phase 4: User Story 2 - 滑鼠互動旋轉 (Priority: P2)

**Goal**: 使用者可透過滑鼠移動來改變觀看地球的角度

**Independent Test**: 移動滑鼠時攝影機視角會跟隨滑鼠位置平滑移動

### Implementation for User Story 2

- [ ] T019 [US2] 實作滑鼠位置追蹤變數 (mouse.x, mouse.y) 和視窗半尺寸計算
- [ ] T020 [US2] 實作 `onDocumentMouseMove` 事件處理器計算目標攝影機位置
- [ ] T021 [US2] 實作 `onDocumentMouseLeave` 事件處理器設定攝影機返回預設位置
- [ ] T022 [US2] 在 animate() 中加入攝影機位置指數衰減插值 (lerp factor 0.05)
- [ ] T023 [US2] 確保攝影機始終朝向球心 (camera.lookAt)
- [ ] T024 [US2] 驗證滑鼠互動流暢度，確認移出視窗後攝影機緩慢回歸

**Checkpoint**: User Story 2 完成，粒子地球支援滑鼠互動視角控制

---

## Phase 5: User Story 3 - 視覺效果調整 (Priority: P3)

**Goal**: 使用者可透過控制面板調整粒子大小和旋轉速度

**Independent Test**: 透過控制面板調整粒子大小，可即時看到粒子尺寸變化

### Implementation for User Story 3

- [ ] T025 [US3] 初始化 lil-gui 控制面板實例
- [ ] T026 [US3] 新增 Point Size 滑桿控制項 (範圍 1.0-10.0, 步進 0.1)
- [ ] T027 [US3] 新增 Rotation Speed 滑桿控制項 (範圍 0.0-0.01, 步進 0.0001)
- [ ] T028 [US3] 新增 Opacity 滑桿控制項 (範圍 0.0-1.0, 步進 0.01)
- [ ] T029 [US3] 驗證 GUI 控制即時更新視覺效果

**Checkpoint**: User Story 3 完成，完整的互動式粒子地球展示

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: 最終優化和完善

- [ ] T030 [P] 新增頁面標題和說明資訊區塊
- [ ] T031 [P] 優化效能確保維持 60 FPS
- [ ] T032 [P] 測試各主流瀏覽器相容性 (Chrome, Firefox, Safari, Edge)
- [ ] T033 程式碼整理和註解完善
- [ ] T034 更新 quickstart.md 確認所有檢查項目完成

---

## Dependencies

```
Phase 1 (Setup)
    │
    ▼
Phase 2 (Foundational)
    │
    ├─────────────────┬─────────────────┐
    ▼                 ▼                 ▼
Phase 3 (US1)   Phase 4 (US2)    Phase 5 (US3)
  P1 MVP         P2 互動          P3 控制
    │                 │                 │
    └─────────────────┴─────────────────┘
                      │
                      ▼
              Phase 6 (Polish)
```

**User Story 相依性**:
- US1 (P1): 無相依，可獨立作為 MVP
- US2 (P2): 相依於 US1 的基礎渲染
- US3 (P3): 相依於 US1 的 uniforms 結構

---

## Parallel Execution Examples

### Phase 1 可平行執行:
```
T002 ─┐
T003 ─┼─ 同時執行 (不同檔案，無相依)
T004 ─┘
```

### Phase 6 可平行執行:
```
T030 ─┐
T031 ─┼─ 同時執行 (獨立優化任務)
T032 ─┘
```

---

## Implementation Strategy

### MVP 範圍 (建議優先交付)
- Phase 1 + Phase 2 + Phase 3 (User Story 1)
- 交付物：可顯示自動旋轉粒子地球的網頁

### 完整功能
- MVP + Phase 4 + Phase 5 + Phase 6
- 交付物：完整互動式粒子地球展示應用

---

## Summary

| 項目 | 數值 |
|------|------|
| **總任務數** | 34 |
| **Phase 1 (Setup)** | 4 |
| **Phase 2 (Foundational)** | 6 |
| **Phase 3 (US1 - MVP)** | 8 |
| **Phase 4 (US2)** | 6 |
| **Phase 5 (US3)** | 5 |
| **Phase 6 (Polish)** | 5 |
| **可平行任務** | 9 |

### 各 User Story 獨立測試標準

| User Story | 獨立測試 |
|------------|----------|
| US1 (P1) | 開啟網頁 → 粒子地球可見且自動旋轉 |
| US2 (P2) | 移動滑鼠 → 攝影機視角跟隨移動 |
| US3 (P3) | 調整 GUI → 粒子效果即時更新 |
