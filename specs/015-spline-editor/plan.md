# Implementation Plan: WebGL Geometry Spline Editor

**Branch**: `015-spline-editor` | **Date**: 2025-12-04 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/015-spline-editor/spec.md`

## Summary

建立一個互動式 Catmull-Rom 曲線編輯器，讓使用者可以在 3D 空間中新增、移除、拖曳控制點來編輯曲線。支援三種曲線類型（uniform、centripetal、chordal），透過 GUI 面板控制曲線顯示和參數，並可匯出控制點座標。技術方案採用單一 HTML 檔案結構，透過 CDN 載入 Three.js 及相關 addons（OrbitControls、TransformControls、lil-gui）。

## Technical Context

**Language/Version**: JavaScript ES6+ (ES Modules)  
**Primary Dependencies**: Three.js r170+ (via CDN), OrbitControls, TransformControls, lil-gui  
**Storage**: N/A (純前端展示，無持久化需求)  
**Testing**: 手動瀏覽器測試 (開啟 HTML 檔案驗證功能)  
**Target Platform**: 現代瀏覽器 (Chrome, Firefox, Safari, Edge) - WebGL 2.0 支援  
**Project Type**: Single HTML file (獨立展示範例)  
**Performance Goals**: 30+ FPS, 100ms 內曲線更新, 3 秒內完成載入  
**Constraints**: 最少 4 個控制點, 最多支援 20+ 控制點不影響效能  
**Scale/Scope**: 3 條曲線 × 200 段/條 = 600 個線段頂點, 4-20 個控制點

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 原則 | 狀態 | 說明 |
|------|------|------|
| 簡單性 (Simplicity) | ✅ Pass | 單一 HTML 檔案，無複雜建置流程 |
| 獨立性 (Self-contained) | ✅ Pass | 僅依賴 CDN 載入的 Three.js 及 addons，可獨立執行 |
| 可測試性 (Testable) | ✅ Pass | 開啟瀏覽器即可手動驗證所有功能 |
| 文件完整性 | ✅ Pass | 規格已完成澄清，無模糊需求 |

**Gate Result**: ✅ PASS - 可進入 Phase 0

## Project Structure

### Documentation (this feature)

```text
specs/015-spline-editor/
├── spec.md              # 功能規格 (已完成)
├── plan.md              # 本檔案 - 實作計畫
├── research.md          # Phase 0 輸出 - 技術研究
├── data-model.md        # Phase 1 輸出 - 資料模型
├── quickstart.md        # Phase 1 輸出 - 快速開始指南
├── contracts/           # Phase 1 輸出 - API 契約
└── checklists/
    └── requirements.md  # 需求檢查清單 (已完成)
```

### Source Code (repository root)

```text
examples/
└── webgl-spline-editor/
    └── index.html       # 單一 HTML 檔案 (內嵌 CSS + JS)
```

**Structure Decision**: 採用極簡結構，單一 HTML 檔案包含所有程式碼，與 Three.js 官方範例結構一致，便於學習和分享。

## Complexity Tracking

> 無違規需要記錄 - 本專案採用最簡單的單檔案結構

---

## Phase 0: Research

### 研究任務

1. **CatmullRomCurve3 API** - 研究 Three.js 內建的 Catmull-Rom 曲線類別，包含 curveType 和 tension 參數
2. **TransformControls 整合** - 研究如何使用 TransformControls 拖曳 3D 物件
3. **OrbitControls 與 TransformControls 協調** - 研究如何在拖曳時停用軌道控制
4. **lil-gui 面板配置** - 研究 GUI 面板的 checkbox、slider、button 配置方式
5. **Raycaster 選取機制** - 研究如何偵測滑鼠點擊選取 3D 物件
6. **BufferGeometry 動態更新** - 研究如何在控制點移動時即時更新曲線幾何

### 研究結果

詳見 [research.md](./research.md)

---

## Phase 1: Design

### 資料模型

詳見 [data-model.md](./data-model.md)

### 快速開始指南

詳見 [quickstart.md](./quickstart.md)

---

## Phase 2: Tasks

> 由 `/speckit.tasks` 命令產生，詳見 [tasks.md](./tasks.md)