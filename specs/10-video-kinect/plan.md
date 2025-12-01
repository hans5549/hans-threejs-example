# Implementation Plan: WebGL Video Kinect 深度影片點雲視覺化

**Branch**: `10-video-kinect` | **Date**: 2025-12-01 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/10-video-kinect/spec.md`

## Summary

實作一個基於 Three.js 的 WebGL Video Kinect 展示範例，將預錄的 Kinect 深度影片轉換為即時 3D 點雲視覺化。技術方案採用自訂 Vertex Shader 解析深度資料並計算 3D 座標，搭配 Fragment Shader 處理顏色和透明度。透過 VideoTexture 作為 Shader 輸入，結合滑鼠互動控制相機和 lil-gui 參數調整面板，提供完整的深度資料探索體驗。

## Technical Context

**Language/Version**: JavaScript ES2020+ (ES Modules)  
**Primary Dependencies**: Three.js r160+ (CDN via unpkg), lil-gui (Three.js addons)  
**Storage**: N/A (純前端展示，影片資源由 CDN 提供)  
**Testing**: 手動視覺測試 + 瀏覽器開發者工具驗證  
**Target Platform**: 現代網頁瀏覽器 (Chrome, Firefox, Safari, Edge 最新版)  
**Project Type**: Single HTML 靜態頁面展示範例  
**Performance Goals**: 30+ fps 流暢渲染，頁面載入後 3 秒內開始顯示點雲  
**Constraints**: 需支援 WebGL 1.0+，影片格式需同時提供 webm 和 mp4 以確保跨瀏覽器相容  
**Scale/Scope**: 單一展示頁面，約 200-250 行程式碼，307,200 個點 (640x480)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 原則 | 狀態 | 說明 |
|------|------|------|
| 簡單性原則 | ✅ PASS | 單一 HTML 檔案，最小化相依性 |
| 可測試性 | ✅ PASS | 可透過視覺確認和瀏覽器工具驗證 |
| 文件完整性 | ✅ PASS | 規格已完整定義所有需求 |
| 範圍控制 | ✅ PASS | 明確的功能邊界和 Edge Cases |
| 效能考量 | ✅ PASS | 已定義 FPS 和延遲指標 |

**Gate Status**: ✅ PASSED - 可繼續進入 Phase 0

## Project Structure

### Documentation (this feature)

```text
specs/10-video-kinect/
├── plan.md              # 本檔案 ✅
├── spec.md              # 功能規格 ✅
├── research.md          # Phase 0 output - 深度轉換技術研究 ✅
├── data-model.md        # Phase 1 output - 資料模型與狀態管理 ✅
├── quickstart.md        # Phase 1 output - 快速入門指南 ✅
├── contracts/           # Phase 1 output - 元件介面定義 ✅
│   └── component-interface.md
├── checklists/
│   └── requirements.md  # 需求檢查清單 ✅
└── tasks.md             # Phase 2 output (by /speckit.tasks) ✅
```

### Source Code (repository root)

```text
examples/
└── webgl-video-kinect/
    └── index.html       # 主要展示頁面（含所有程式碼）
```

**Structure Decision**: 採用單一 HTML 檔案結構，符合 Three.js 官方範例慣例。所有 JavaScript、CSS 和 GLSL shader 程式碼都內嵌於 index.html 中。影片資源使用 Three.js 官方提供的 Kinect 深度影片 (textures/kinect.webm, textures/kinect.mp4)。

## Complexity Tracking

> 無違規項目，不需要複雜度追蹤。

## Phase 0: Research Summary ✅ COMPLETE

**Generated Artifacts:**
- [research.md](./research.md) - 詳細技術研究文件

### 核心技術決策

| 決策 | 選擇 | 理由 |
|------|------|------|
| 深度轉換方式 | 自訂 Vertex Shader | 利用 GPU 並行計算，每個頂點獨立轉換 |
| 點雲渲染 | THREE.Points | 專為大量點渲染設計的 Three.js 類別 |
| 影片輸入 | THREE.VideoTexture | 自動同步影片幀到紋理，支援即時更新 |
| 深度解析 | RGB 平均值 | Kinect 深度影片將深度編碼為灰階值 |
| 相機類型 | PerspectiveCamera | 需要 3D 透視效果觀察點雲 |
| 混合模式 | AdditiveBlending | 產生發光效果，增強視覺層次感 |

### Kinect 視野參數

```
水平 FOV: ~58° → XtoZ = tan(58°/2) * 2 ≈ 1.11146
垂直 FOV: ~45° → YtoZ = tan(45°/2) * 2 ≈ 0.83359

深度範圍:
- 近裁切預設: 850mm
- 遠裁切預設: 4000mm
```

### 深度到 3D 座標轉換公式

```glsl
// 從深度值計算實際深度距離
float depth = (color.r + color.g + color.b) / 3.0;
float z = (1.0 - depth) * (farClipping - nearClipping) + nearClipping;

// 使用 Kinect 視野參數計算 X, Y 座標
float x = (pixelX / width - 0.5) * z * XtoZ;
float y = (pixelY / height - 0.5) * z * YtoZ;
```

### 關鍵實作考量

1. **BufferGeometry 建立**: 預先建立 640x480 = 307,200 個頂點，僅儲存像素座標
2. **Shader 動態計算**: 在 Vertex Shader 中讀取 VideoTexture，計算實際 3D 位置
3. **相機跟隨滑鼠**: 使用 lerp 插值實現平滑的相機移動
4. **GUI 參數綁定**: 直接綁定 ShaderMaterial uniforms 實現即時更新

## Phase 1: Design Decisions ✅ COMPLETE

**Generated Artifacts:**
- [data-model.md](./data-model.md) - Shader uniforms, 幾何資料、狀態管理定義
- [quickstart.md](./quickstart.md) - 快速上手指南與實作步驟
- [contracts/component-interface.md](./contracts/component-interface.md) - 元件介面契約

### 架構概覽

```
┌─────────────────────────────────────────────────────────────────┐
│                         HTML Document                            │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐                                              │
│  │ <video>       │ (hidden, autoplay, loop, muted)              │
│  │ kinect.webm   │                                              │
│  │ kinect.mp4    │                                              │
│  └───────┬───────┘                                              │
│          │                                                       │
│          ▼                                                       │
│  ┌───────────────┐     ┌─────────────────────────────────────┐ │
│  │ VideoTexture  │────▶│        Vertex Shader                 │ │
│  └───────────────┘     │  - 讀取深度值 (RGB avg)              │ │
│                        │  - 計算 3D 座標 (Kinect FOV)         │ │
│                        │  - 設定 gl_PointSize                 │ │
│                        └──────────────┬──────────────────────┘ │
│                                       │                         │
│  ┌───────────────┐                    ▼                         │
│  │ Uniforms      │     ┌─────────────────────────────────────┐ │
│  │ - map         │────▶│        Fragment Shader               │ │
│  │ - width/height│     │  - 讀取顏色                          │ │
│  │ - near/far    │     │  - 設定透明度 0.2                    │ │
│  │ - pointSize   │     └──────────────┬──────────────────────┘ │
│  │ - zOffset     │                    │                         │
│  └───────────────┘                    ▼                         │
│                        ┌─────────────────────────────────────┐ │
│                        │        ShaderMaterial                │ │
│                        │  - AdditiveBlending                  │ │
│                        │  - depthTest: false                  │ │
│                        │  - transparent: true                 │ │
│                        └──────────────┬──────────────────────┘ │
│                                       │                         │
│  ┌───────────────┐                    ▼                         │
│  │ BufferGeometry│     ┌─────────────────────────────────────┐ │
│  │ 640x480 points│────▶│          THREE.Points                │ │
│  └───────────────┘     └──────────────┬──────────────────────┘ │
│                                       │                         │
│                                       ▼                         │
│  ┌───────────────┐     ┌─────────────────────────────────────┐ │
│  │ Mouse Input   │────▶│     PerspectiveCamera                │ │
│  │ (smoothed)    │     │     lookAt(center)                   │ │
│  └───────────────┘     └──────────────┬──────────────────────┘ │
│                                       │                         │
│  ┌───────────────┐                    ▼                         │
│  │ GUI (lil-gui) │     ┌─────────────────────────────────────┐ │
│  │ Parameter     │────▶│        WebGLRenderer                 │ │
│  │ Controls      │     │        setAnimationLoop()            │ │
│  └───────────────┘     └─────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 實作優先順序

| 優先級 | 元件 | 說明 | 對應 User Story |
|--------|------|------|-----------------|
| P1 | Video 元素 | HTML5 video 載入 Kinect 深度影片 | US1 |
| P1 | VideoTexture | 建立動態紋理供 Shader 使用 | US1 |
| P1 | BufferGeometry | 建立 307,200 個頂點的點雲結構 | US1 |
| P1 | Vertex Shader | 深度到 3D 座標轉換邏輯 | US1 |
| P1 | Fragment Shader | 顏色和透明度處理 | US1 |
| P1 | ShaderMaterial | 整合 Shader 和 Uniforms | US1 |
| P1 | Scene + Renderer | 基礎渲染管線初始化 | US1 |
| P2 | Mouse Tracking | 滑鼠位置追蹤和平滑處理 | US2 |
| P2 | Camera Animation | 相機位置插值和 lookAt | US2 |
| P3 | GUI Controller | lil-gui 參數控制面板 | US3 |
| P3 | Window Resize | 視窗調整處理 | Edge Case |

### Shader Uniforms 規格

| Uniform | 類型 | 預設值 | 說明 |
|---------|------|--------|------|
| map | sampler2D | VideoTexture | 深度影片紋理 |
| width | float | 640 | 影片寬度 (像素) |
| height | float | 480 | 影片高度 (像素) |
| nearClipping | float | 850 | 近裁切距離 (mm) |
| farClipping | float | 4000 | 遠裁切距離 (mm) |
| pointSize | float | 2 | 點大小 (像素) |
| zOffset | float | 1000 | Z 軸偏移 (mm) |

## Next Steps

1. 執行 `/speckit.tasks` 產生詳細任務清單
2. 依據任務清單進行實作
3. 完成後執行視覺驗證測試，確認：
   - 點雲正確反映深度變化
   - 滑鼠控制相機流暢
   - GUI 參數調整即時生效
