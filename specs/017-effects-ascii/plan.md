# Implementation Plan: WebGL Effects ASCII

**Branch**: `017-effects-ascii` | **Date**: 2025-01-27 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/017-effects-ascii/spec.md`

## Summary

實作 Three.js WebGL Effects ASCII 範例，將 3D 場景（含彈跳球體和平面）透過 AsciiEffect 轉換為 ASCII 字元藝術渲染。使用 TrackballControls 提供相機互動控制，並支援視窗大小自適應。

## Technical Context

**Language/Version**: JavaScript ES6+  
**Primary Dependencies**: Three.js r170.0 (via CDN), AsciiEffect, TrackballControls  
**Storage**: N/A  
**Testing**: Manual browser testing  
**Target Platform**: Modern web browsers with WebGL support  
**Project Type**: Single HTML file example  
**Performance Goals**: 30+ fps 流暢渲染  
**Constraints**: 單一 HTML 檔案，使用 CDN 載入 Three.js  
**Scale/Scope**: 單一展示範例

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| 專案結構 | ✅ PASS | 遵循現有 examples/ 目錄結構 |
| 程式碼品質 | ✅ PASS | 使用 ES6 模組語法，程式碼清晰易讀 |
| 相依性管理 | ✅ PASS | 使用 CDN 載入 Three.js，無額外相依性 |
| 文件完整性 | ✅ PASS | 包含完整規格文件和實作計畫 |

## Project Structure

### Documentation (this feature)

```text
specs/017-effects-ascii/
├── plan.md              # This file
├── research.md          # Phase 0 output - AsciiEffect API 研究
├── data-model.md        # Phase 1 output - 資料模型
├── quickstart.md        # Phase 1 output - 快速入門指南
├── contracts/           # Phase 1 output - API 契約
└── tasks.md             # Phase 2 output - 任務清單
```

### Source Code (repository root)

```text
examples/
└── webgl-effects-ascii/
    └── index.html       # 實作目標檔案
```

**Structure Decision**: 採用單一 HTML 檔案結構，與現有 examples/ 目錄中的其他範例保持一致。所有程式碼（HTML、CSS、JavaScript）都包含在單一 index.html 檔案中。

## Complexity Tracking

> 無違規需要說明 - 本功能遵循簡單的單檔案架構。

## File Organization

| File | Purpose | Status |
|------|---------|--------|
| `examples/webgl-effects-ascii/index.html` | 主要實作檔案 | ✅ Complete |

## Implementation Notes

### AsciiEffect 設定
- 字元集：` .:-+*=%@#`
- 選項：`{ invert: true }` 實現白字黑底效果
- DOM 樣式：`color: white`, `backgroundColor: black`

### 場景配置
- 相機：PerspectiveCamera(FOV 70, y=150, z=500)
- 球體：SphereGeometry(200, 20, 10)
- 平面：PlaneGeometry(400, 400), y=-200
- 光源：兩個 PointLight

### 動畫設計
- 球體垂直位移：使用 Math.abs(Math.sin(start * 2)) * 150
- 球體旋轉：x 軸和 z 軸持續旋轉
