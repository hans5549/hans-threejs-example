# Implementation Plan: WebGPU Custom Lighting Model

**Branch**: `13-webgpu-lights-custom` | **Date**: 2025-12-01 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/13-webgpu-lights-custom/spec.md`

## Summary

實作 Three.js WebGPU 自訂光照模型範例。使用 WebGPURenderer 搭配 TSL (Three Shading Language) 建立自訂光照計算邏輯，展示選擇性光源 (Selective Lights) 如何應用於 50 萬粒子的點雲視覺化，並使用三個動態彩色點光源產生即時光影效果。

## Technical Context

**Language/Version**: JavaScript ES2022+ (ES Modules)  
**Primary Dependencies**: Three.js r170+ (WebGPU build), TSL (Three Shading Language)  
**Storage**: N/A (前端靜態頁面)  
**Testing**: 手動瀏覽器測試 + Console 檢查  
**Target Platform**: 支援 WebGPU 的現代瀏覽器 (Chrome 113+, Edge 113+)  
**Project Type**: Single HTML file  
**Performance Goals**: 60 FPS 目標，最低可接受 30 FPS（50 萬粒子）  
**Constraints**: 僅支援 WebGPU，不支援時顯示靜態圖片  
**Scale/Scope**: 單頁展示範例，無使用者資料

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

本專案使用通用模板 constitution，未定義特定限制規則。

| Gate | Status | Notes |
|------|--------|-------|
| 專案複雜度 | ✅ PASS | 單一 HTML 檔案，符合現有範例結構 |
| 依賴數量 | ✅ PASS | 僅使用 Three.js CDN，無額外依賴 |
| 測試策略 | ✅ PASS | 前端展示範例，手動測試為主 |

## Project Structure

### Documentation (this feature)

```text
specs/13-webgpu-lights-custom/
├── plan.md              # This file
├── research.md          # ✅ Phase 0 output
├── data-model.md        # ✅ Phase 1 output
├── quickstart.md        # ✅ Phase 1 output
├── contracts/           # ✅ Phase 1 output
│   └── component-interface.md
├── tasks.md             # Phase 2 output (/speckit.tasks)
├── spec.md              # Feature specification
└── checklists/
    └── requirements.md
```

### Source Code (repository root)

```text
examples/
├── webgpu-volume-caustics/
│   └── index.html
├── ... (other examples)
└── webgpu-lights-custom/           # NEW
    ├── index.html                  # 完整單頁應用
    └── assets/
        └── fallback.png            # WebGPU 不支援時的靜態圖片
```

**Structure Decision**: 遵循現有 `examples/` 目錄結構，每個範例為獨立資料夾，包含單一 `index.html` 檔案。使用 CDN import map 載入 Three.js，無需本地 node_modules。新增 `assets/` 子目錄存放 fallback 圖片。

## Implementation Phases

### Phase 1: 基礎架構 (Priority: P0)

建立基本 HTML 結構、WebGPU 偵測、錯誤處理 UI。

**Tasks**:
1. 建立 `examples/webgpu-lights-custom/index.html` 基本結構
2. 實作 WebGPU 支援偵測邏輯
3. 建立不支援時的 fallback UI（靜態圖片 + 訊息）
4. 設定 ES Modules import map
5. 建立 loading 和 info UI 元素

**產出**:
- 可在瀏覽器開啟的基本頁面
- WebGPU 支援/不支援分支邏輯

**驗證**:
- Chrome 中顯示空白場景（或初始化訊息）
- Safari 中顯示 fallback 靜態圖片

**對應需求**: FR-001

---

### Phase 2: 場景與渲染器 (Priority: P0)

建立 Three.js 場景、相機、WebGPU 渲染器。

**Tasks**:
1. 初始化 WebGPURenderer（antialias: true）
2. 建立 PerspectiveCamera（FOV: 70, z: 1.5）
3. 建立 Scene（黑色背景 0x000000）
4. 實作 resize 事件處理
5. 建立動畫迴圈 (setAnimationLoop)

**產出**:
- 可運行的 WebGPU 渲染管線
- 響應式畫面調整

**驗證**:
- 畫面顯示黑色背景
- 調整視窗大小時畫面正確調整

**對應需求**: FR-001, FR-009, FR-010

---

### Phase 3: 光源系統 (Priority: P1 - Core Feature)

建立三個動態點光源及其視覺指示器。

**Tasks**:
1. 建立共用 SphereGeometry（半徑 0.02）
2. 建立 `addLight(hexColor)` 工廠函式
3. 實作 NodeMaterial 自發光材質（使用空 lightsNode）
4. 建立三個 PointLight（橙 0xffaa00、藍 0x0040ff、綠 0x80ff80）
5. 設定光源強度（0.1）和距離（1）
6. 將小球 mesh 附加到光源上

**產出**:
- 三個彩色光源，各自帶有視覺指示小球
- 小球使用自發光效果不受其他光源影響

**驗證**:
- 三個不同顏色的小球可見
- 小球顏色為純色（不受其他光源影響）

**對應需求**: FR-003, FR-006, FR-011

---

### Phase 4: 自訂光照模型 (Priority: P1 - Core Feature)

實作 CustomLightingModel 類別和選擇性光源節點。

**Tasks**:
1. 建立 `CustomLightingModel` 類別繼承 `THREE.LightingModel`
2. 實作 `direct()` 方法：將 lightColor 加到 reflectedLight.directDiffuse
3. 建立選擇性光源節點 `lights([light1, light2, light3])`
4. 建立光照模型上下文 `allLightsNode.context({ lightingModel })`

**產出**:
- 可用於材質的自訂光照模型
- 將三個光源綁定為選擇性光源

**驗證**:
- 程式碼無錯誤執行

**對應需求**: FR-002, FR-004

---

### Phase 5: 點雲建立 (Priority: P1 - Core Feature)

產生 50 萬個隨機粒子並套用自訂光照。

**Tasks**:
1. 產生 50 萬個隨機 Vector3 點（範圍 ±1.5）
2. 建立 BufferGeometry.setFromPoints()
3. 建立 PointsNodeMaterial
4. 將光照模型上下文設定為 `materialPoints.lightsNode`
5. 建立 Points 物件並加入場景

**產出**:
- 50 萬粒子的點雲，使用自訂光照模型
- 粒子會根據光源距離和顏色被照亮

**驗證**:
- 點雲清晰可見
- 光源附近的粒子顯示該光源顏色

**對應需求**: FR-005, FR-004

---

### Phase 6: 動畫與互動 (Priority: P2)

實作光源動畫和相機控制。

**Tasks**:
1. 實作 `animate()` 函式
2. 使用三角函數計算三個光源的位置（scale: 0.5）
3. 為每個光源設定不同的頻率參數
4. 實作場景緩慢旋轉（scene.rotation.y）
5. 整合 OrbitControls（minDistance: 0, maxDistance: 4）

**產出**:
- 三個光源以不同軌跡動態移動
- 使用者可自由旋轉縮放視角

**驗證**:
- 光源持續移動且軌跡不同
- 滑鼠拖曳可旋轉、滾輪可縮放

**對應需求**: FR-007, FR-008

---

### Phase 7: UI 與最終調整 (Priority: P3)

完善 UI 元素和錯誤處理。

**Tasks**:
1. 建立 info header（標題和說明）
2. 建立 loading 動畫
3. 隱藏 loading 當場景就緒
4. 測試並調整效能
5. 驗證所有成功標準

**產出**:
- 完整的使用者介面
- 穩定流暢的動畫效果

**驗證**:
- 所有 Success Criteria (SC-001 到 SC-006) 通過

**對應需求**: FR-009, SC-001 ~ SC-006

## Complexity Tracking

> 無違規需要記錄
