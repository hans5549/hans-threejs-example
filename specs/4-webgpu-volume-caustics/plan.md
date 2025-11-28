# Implementation Plan: WebGPU Volume Caustics

**Branch**: `4-webgpu-volume-caustics` | **Date**: 2025-11-28 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/4-webgpu-volume-caustics/spec.md`

## Summary

實作 Three.js WebGPU 即時體積焦散效果範例。使用 WebGPURenderer 搭配 TSL (Three Shading Language) 自訂著色器，在透明玻璃鴨子模型上產生焦散光影效果，並結合體積光 (Volumetric Lighting) 和後處理 Bloom 效果，展示 WebGPU 的進階渲染能力。

## Technical Context

**Language/Version**: JavaScript ES2022+ (ES Modules)  
**Primary Dependencies**: Three.js r170+ (WebGPU build), TSL (Three Shading Language)  
**Storage**: N/A (前端靜態頁面)  
**Testing**: 手動瀏覽器測試 + Console 檢查  
**Target Platform**: 支援 WebGPU 的現代瀏覽器 (Chrome 113+, Edge 113+)  
**Project Type**: Single HTML file  
**Performance Goals**: 60 FPS 目標，最低可接受 30 FPS  
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
specs/4-webgpu-volume-caustics/
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
├── css3d-periodic-table/
│   └── index.html
├── interactive-raycasting-points/
│   └── index.html
├── unreal-bloom-postprocessing/
│   └── index.html
└── webgpu-volume-caustics/        # NEW
    └── index.html                 # 完整單頁應用
```

**Structure Decision**: 遵循現有 `examples/` 目錄結構，每個範例為獨立資料夾，包含單一 `index.html` 檔案。使用 CDN import map 載入 Three.js，無需本地 node_modules。

## Implementation Phases

### Phase 1: 基礎架構 (Priority: P0)

建立基本 HTML 結構、WebGPU 偵測、錯誤處理 UI。

**Tasks**:
1. 建立 `examples/webgpu-volume-caustics/index.html` 基本結構
2. 實作 WebGPU 支援偵測邏輯
3. 建立不支援時的 fallback UI（靜態圖片 + 訊息）
4. 設定 ES Modules import map

**產出**:
- 可在瀏覽器開啟的基本頁面
- WebGPU 支援/不支援分支邏輯

**驗證**:
- Chrome 中顯示空白場景（或初始化訊息）
- Safari 中顯示 fallback UI

---

### Phase 2: 場景建立 (Priority: P0)

建立 Three.js 場景、相機、渲染器、光源。

**Tasks**:
1. 初始化 WebGPURenderer
2. 建立 PerspectiveCamera 並設定位置
3. 建立 Scene
4. 配置 SpotLight 並啟用 HDR 陰影
5. 建立地面 PlaneGeometry 並接收陰影
6. 實作 OrbitControls 相機控制

**產出**:
- 可見的聚光燈照射地面場景
- 可互動的相機控制

**驗證**:
- 滑鼠可旋轉/縮放視角
- 地面可見光源照射效果

---

### Phase 3: 模型載入 (Priority: P1)

載入並配置透明鴨子模型。

**Tasks**:
1. 配置 GLTFLoader + DRACOLoader
2. 載入 duck.glb 模型
3. 建立 MeshPhysicalNodeMaterial 透明材質
4. 設定 transmission、ior、thickness 等屬性
5. 啟用模型陰影投射

**產出**:
- 場景中顯示透明鴨子模型
- 模型具有玻璃材質外觀

**驗證**:
- 鴨子清晰可見且具透明效果
- 無模型載入錯誤

---

### Phase 4: 焦散效果 (Priority: P1 - Core Feature)

實作 TSL 自訂焦散著色器。

**Tasks**:
1. 載入焦散紋理 (Caustic_Free.jpg)
2. 實作 `causticEffect` TSL 函式
3. 實作 `refract` 折射計算
4. 實作色散效果 (chromatic aberration)
5. 將 causticEffect 設定為 `castShadowNode`
6. 實作 `emissiveNode` 背光散射效果

**產出**:
- 地面上可見動態焦散光斑
- 焦散具有彩虹色散效果

**驗證**:
- 焦散圖案清晰可見
- 可見 RGB 分離的色散效果

---

### Phase 5: 體積光效果 (Priority: P2)

實作體積光 (Volumetric Lighting)。

**Tasks**:
1. 產生 3D 噪聲紋理 (Data3DTexture)
2. 建立 VolumeNodeMaterial
3. 實作 `scatteringNode` 散射函式
4. 配置體積光 layer 系統
5. 建立體積光 BoxGeometry mesh

**產出**:
- 場景中可見光束穿透霧氣效果
- 霧氣具有動態流動感

**驗證**:
- 體積光在聚光燈方向可見
- 煙霧隨時間緩慢移動

---

### Phase 6: 後處理 (Priority: P2)

配置後處理管線和 Bloom 效果。

**Tasks**:
1. 建立 PostProcessing 實例
2. 配置 scenePass 場景渲染
3. 配置 volumetricPass 體積光渲染
4. 配置 bloomPass 光暈效果
5. 合成最終輸出

**產出**:
- 完整的後處理管線
- 光源周圍具有 Bloom 光暈

**驗證**:
- 體積光具有柔和光暈
- 無明顯的渲染瑕疵

---

### Phase 7: 動畫與最終調整 (Priority: P3)

加入動畫循環和響應式調整。

**Tasks**:
1. 實作 animate 函式
2. 加入鴨子自動旋轉
3. 實作 onWindowResize 處理
4. 最終效能調整與測試

**產出**:
- 完整可運行的範例
- 響應式視窗調整

**驗證**:
- 鴨子持續旋轉
- 調整視窗大小後畫面正確適應
- FPS 維持在可接受範圍

## Dependencies Graph

```
Phase 1 (基礎架構)
    ↓
Phase 2 (場景建立)
    ↓
Phase 3 (模型載入)
    ↓
Phase 4 (焦散效果) ←── Core Feature
    ↓
Phase 5 (體積光效果)
    ↓
Phase 6 (後處理)
    ↓
Phase 7 (動畫與最終調整)
```

## Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| WebGPU API 變更 | Low | High | 使用穩定版 Three.js r170 |
| 效能不足 | Medium | Medium | 降低體積光解析度、減少煙霧複雜度 |
| 模型載入失敗 | Low | High | 提供替代 CDN 路徑 |
| TSL 語法錯誤 | Medium | Medium | 參考官方範例程式碼 |

## External Resources

- 官方範例: https://threejs.org/examples/#webgpu_volume_caustics
- 官方原始碼: https://github.com/mrdoob/three.js/blob/dev/examples/webgpu_volume_caustics.html
- TSL Wiki: https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language
- Duck Model: https://cdn.jsdelivr.net/npm/three@0.170.0/examples/models/gltf/duck.glb

## Next Steps

執行 `/speckit.tasks` 指令以產生詳細的任務清單 (`tasks.md`)。
