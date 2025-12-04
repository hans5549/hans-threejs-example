# Implementation Plan: WebGL 文字描邊效果

**Branch**: `018-text-stroke` | **Date**: 2025-12-04 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/018-text-stroke/spec.md`

## Summary

實作 Three.js WebGL 文字描邊範例，使用 Font.generateShapes 產生文字形狀、ShapeGeometry 建立填充幾何體、SVGLoader.pointsToStroke 產生描邊幾何體，支援三種文字方向（左到右、右到左、上到下）。技術方案已透過研究驗證，採用 fflate 解壓縮字體檔案、MeshBasicMaterial 實現半透明填充和實心描邊的視覺效果。

## Technical Context

**Language/Version**: JavaScript (ES6 Modules)  
**Primary Dependencies**: Three.js 0.170.0 (CDN)  
**Storage**: N/A（純前端展示）  
**Testing**: 手動視覺測試  
**Target Platform**: 現代瀏覽器（支援 WebGL 和 ES6 模組）  
**Project Type**: Single HTML file（獨立範例）  
**Performance Goals**: 60 fps 互動、3 秒內完成載入  
**Constraints**: 無額外建構工具、純 CDN 引用  
**Scale/Scope**: 單一展示頁面

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 檢查項目 | 狀態 | 備註 |
|----------|------|------|
| 獨立範例 | ✅ PASS | 不影響其他範例 |
| CDN 引用 | ✅ PASS | 使用 jsdelivr CDN |
| 無額外相依 | ✅ PASS | 僅使用 Three.js 內建模組 |
| 檔案結構一致 | ✅ PASS | 遵循 examples/[name]/index.html 模式 |

**Post-Design Re-check**: ✅ PASS - 設計符合專案規範

## Project Structure

### Documentation (this feature)

```text
specs/018-text-stroke/
├── plan.md              # 本檔案（/speckit.plan 輸出）
├── research.md          # Phase 0 研究結果
├── data-model.md        # Phase 1 資料模型
├── quickstart.md        # Phase 1 快速入門指南
├── contracts/           # Phase 1 API 合約
│   └── api.md
├── checklists/          # 檢查清單
│   └── requirements.md
└── tasks.md             # Phase 2 任務清單（由 /speckit.tasks 建立）
```

### Source Code (repository root)

```text
examples/
└── webgl-geometry-text-stroke/
    └── index.html       # 完整範例實作

fonts/
└── MPLUSRounded1c/
    └── MPLUSRounded1c-Regular.typeface.json.zip  # 字體檔案（需下載）
```

**Structure Decision**: 採用專案標準的單一 HTML 範例結構，與其他 examples/ 目錄下的範例保持一致。字體檔案需從 Three.js 官方範例下載並放置於 fonts/ 目錄。

## Implementation Phases

### Phase 1: 基礎架構
- 建立 HTML 檔案結構
- 設定 importmap 和基本樣式
- 建立 Three.js 場景、相機、渲染器

### Phase 2: 字體載入
- 實作字體檔案載入邏輯
- 使用 fflate 解壓縮 zip
- 解析 JSON 並建立 Font 物件

### Phase 3: 文字描邊生成
- 實作 generateStrokeText 函式
- 建立填充幾何體和網格
- 建立描邊幾何體和網格
- 處理孔洞形狀

### Phase 4: 多方向文字
- 建立英文文字（ltr）
- 建立希伯來文文字（rtl）
- 建立中文文字（tb）
- 定位三組文字

### Phase 5: 互動與完善
- 設定 OrbitControls
- 實作視窗大小調整處理
- 最終測試與調整

## Complexity Tracking

> 無違規項目，無需記錄複雜度追蹤。

## Next Steps

執行 `/speckit.tasks` 產生詳細任務清單。

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
