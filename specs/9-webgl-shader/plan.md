# Implementation Plan: WebGL Shader 動態視覺效果

**Branch**: `9-webgl-shader` | **Date**: 2025-12-01 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/9-webgl-shader/spec.md`

## Summary

實作一個基於 Three.js 的 WebGL Shader 展示範例，使用自訂 GLSL shader (Monjori) 建立動態視覺效果。技術方案採用正交相機搭配全螢幕平面幾何體，透過 ShaderMaterial 將時間參數傳遞給 fragment shader 以產生即時動態的程式化視覺藝術效果。

## Technical Context

**Language/Version**: JavaScript ES2020+ (ES Modules)  
**Primary Dependencies**: Three.js r160+ (CDN via unpkg)  
**Storage**: N/A (純前端展示，無資料持久化需求)  
**Testing**: 手動視覺測試 + 瀏覽器開發者工具驗證  
**Target Platform**: 現代網頁瀏覽器 (Chrome, Firefox, Safari, Edge 最新版)  
**Project Type**: Single HTML 靜態頁面展示範例  
**Performance Goals**: 60fps 流暢渲染，頁面載入後 2 秒內開始顯示  
**Constraints**: 需支援 WebGL 1.0+，無外部相依性（僅 CDN 載入 Three.js）  
**Scale/Scope**: 單一展示頁面，約 150-200 行程式碼

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 原則 | 狀態 | 說明 |
|------|------|------|
| 簡單性原則 | ✅ PASS | 單一 HTML 檔案，最小化相依性 |
| 可測試性 | ✅ PASS | 可透過視覺確認和瀏覽器工具驗證 |
| 文件完整性 | ✅ PASS | 規格已完整定義所有需求 |
| 範圍控制 | ✅ PASS | 明確的 Out of Scope 定義 |

**Gate Status**: ✅ PASSED - 可繼續進入 Phase 0

## Project Structure

### Documentation (this feature)

```text
specs/9-webgl-shader/
├── plan.md              # 本檔案
├── research.md          # Phase 0 output - GLSL shader 技術研究
├── data-model.md        # Phase 1 output - 資料模型與狀態管理
├── quickstart.md        # Phase 1 output - 快速入門指南
├── contracts/           # Phase 1 output - 元件介面定義
│   └── component-interface.md
├── checklists/
│   └── requirements.md  # 需求檢查清單
└── tasks.md             # Phase 2 output (by /speckit.tasks)
```

### Source Code (repository root)

```text
examples/
└── webgl-shader/
    └── index.html       # 主要展示頁面（含所有程式碼）
```

**Structure Decision**: 採用單一 HTML 檔案結構，符合 Three.js 官方範例慣例。所有 JavaScript、CSS 和 GLSL shader 程式碼都內嵌於 index.html 中，便於學習和參考。

## Complexity Tracking

> 無違規項目，不需要複雜度追蹤。

## Phase 0: Research Summary

### 核心技術決策

| 決策 | 選擇 | 理由 |
|------|------|------|
| Shader 類型 | Fragment Shader (Monjori) | 經典程式化視覺藝術效果，僅依賴時間參數 |
| 相機類型 | OrthographicCamera | 適用於 2D 全螢幕渲染，無透視變形 |
| 幾何體 | PlaneGeometry(2, 2) | 精確覆蓋正交相機的可視範圍 |
| 動畫方式 | setAnimationLoop | Three.js 推薦的動畫迴圈 API |
| 時間來源 | performance.now() | 高精度時間戳，適用於動畫 |

### 關鍵實作考量

1. **Shader 嵌入方式**: 使用 `<script type="x-shader/x-vertex">` 和 `<script type="x-shader/x-fragment">` 標籤
2. **Uniform 傳遞**: 透過 `uniforms.time.value` 每幀更新時間值
3. **視窗調整**: 僅需更新 renderer 尺寸，正交相機參數固定不變

## Phase 1: Design Decisions

### 架構概覽

```
┌─────────────────────────────────────────────────────────┐
│                    HTML Document                         │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │ Vertex      │  │ Fragment    │  │ JavaScript      │ │
│  │ Shader      │  │ Shader      │  │ Module          │ │
│  │ (GLSL)      │  │ (GLSL)      │  │                 │ │
│  └──────┬──────┘  └──────┬──────┘  │  ┌───────────┐  │ │
│         │                │         │  │ init()    │  │ │
│         └────────┬───────┘         │  │ animate() │  │ │
│                  │                 │  │ resize()  │  │ │
│                  ▼                 │  └───────────┘  │ │
│         ┌────────────────┐        │        │        │ │
│         │ ShaderMaterial │◄───────┼────────┘        │ │
│         └───────┬────────┘        │                 │ │
│                 │                 │                 │ │
│                 ▼                 │                 │ │
│         ┌────────────────┐        │                 │ │
│         │     Mesh       │        │                 │ │
│         │ (Plane + Mat)  │        │                 │ │
│         └───────┬────────┘        │                 │ │
│                 │                 │                 │ │
│                 ▼                 │                 │ │
│         ┌────────────────┐        │                 │ │
│         │    Scene       │        │                 │ │
│         └───────┬────────┘        │                 │ │
│                 │                 │                 │ │
│                 ▼                 │                 │ │
│  ┌──────────────────────────────┐ │                 │ │
│  │        WebGLRenderer          │◄─────────────────┘ │
│  └──────────────────────────────┘                     │
└─────────────────────────────────────────────────────────┘
```

### 實作優先順序

| 優先級 | 元件 | 說明 |
|--------|------|------|
| P1 | 基礎渲染管線 | Scene, Camera, Renderer 初始化 |
| P1 | Shader 整合 | Vertex/Fragment shader + ShaderMaterial |
| P1 | 動畫迴圈 | time uniform 更新與渲染 |
| P2 | 視窗調整 | resize 事件處理 |
| P3 | UI 資訊 | 標題和來源連結 |

## Next Steps

1. 執行 `/speckit.tasks` 產生詳細任務清單
2. 依據任務清單進行實作
3. 完成後執行視覺驗證測試
