# Implementation Plan: BufferGeometry DrawRange 互動式粒子網絡

**Branch**: `7-buffergeometry-drawrange` | **Date**: 2025-11-28 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/7-buffergeometry-drawrange/spec.md`

## Summary

實作一個互動式 3D 粒子網絡視覺化系統，展示 Three.js 的 BufferGeometry 和 drawRange 動態渲染技術。使用者可觀看最多 1000 個粒子在立方體邊界內移動，當粒子距離足夠接近時會自動產生透明度隨距離變化的連線。提供 GUI 控制面板讓使用者調整各項參數，並支援軌道控制探索場景。

## Technical Context

**Language/Version**: JavaScript ES6+ Modules  
**Primary Dependencies**: Three.js r160+, lil-gui, Stats.js  
**Storage**: N/A（純前端應用）  
**Testing**: 手動瀏覽器測試  
**Target Platform**: 現代瀏覽器（Chrome, Firefox, Safari, Edge），需支援 WebGL  
**Project Type**: Single HTML file（遵循專案現有結構）  
**Performance Goals**: ≥30 FPS @ 500 粒子, ≥60 FPS @ 200 粒子  
**Constraints**: 最大 1000 粒子，攝影機距離 1000-3000 單位  
**Scale/Scope**: 單一展示頁面

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 原則 | 狀態 | 說明 |
|------|------|------|
| 結構一致性 | ✅ Pass | 遵循現有 `examples/` 目錄結構 |
| 相依性管理 | ✅ Pass | 使用 CDN import map，與現有範例一致 |
| 效能要求 | ✅ Pass | 已定義明確的 FPS 目標和粒子上限 |
| 無額外複雜度 | ✅ Pass | 單一 HTML 檔案，無額外建置流程 |

## Project Structure

### Documentation (this feature)

```text
specs/7-buffergeometry-drawrange/
├── spec.md              # 功能規格
├── plan.md              # 本檔案
├── research.md          # Phase 0 研究輸出
├── data-model.md        # Phase 1 資料模型
├── quickstart.md        # Phase 1 快速入門指南
├── contracts/           # Phase 1 元件介面
│   └── component-interface.md
├── tasks.md             # Phase 2 任務清單
└── checklists/
    └── requirements.md  # 規格驗證檢查清單
```

### Source Code (repository root)

```text
examples/
└── buffergeometry-drawrange/
    └── index.html       # 主要展示頁面（含所有 JavaScript）
```

**Structure Decision**: 使用單一 HTML 檔案結構，與專案中其他範例（如 `interactive-raycasting-points/`、`unreal-bloom-postprocessing/`）保持一致。所有 JavaScript 程式碼內嵌於 HTML 的 `<script type="module">` 標籤中，透過 CDN import map 載入 Three.js 相依套件。

## Complexity Tracking

> **無違規需要記錄** - 本功能完全符合專案結構規範。

---

## Phase 0: 研究與調查

### 待研究項目

1. **BufferGeometry 動態更新最佳實踐**
   - DynamicDrawUsage vs StaticDrawUsage 差異
   - needsUpdate 屬性使用時機
   - setDrawRange 方法的效能影響

2. **粒子間距離計算最佳化**
   - O(n²) 演算法在 1000 粒子情況下的效能
   - 是否需要空間分割優化（如 Octree）

3. **lil-gui 整合模式**
   - 與現有專案範例的 GUI 使用方式對齊

### 研究輸出

→ 請參閱 [research.md](research.md)

---

## Phase 1: 設計與合約

### 資料模型設計

→ 請參閱 [data-model.md](data-model.md)

### 元件介面

→ 請參閱 [contracts/component-interface.md](contracts/component-interface.md)

### 快速入門

→ 請參閱 [quickstart.md](quickstart.md)

---

## Phase 2: 任務規劃

→ 由 `/speckit.tasks` 命令產生 [tasks.md](tasks.md)
