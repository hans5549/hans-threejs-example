# Implementation Plan: Three.js 互動式點雲 Raycasting 範例

**Branch**: `1-interactive-raycasting-points` | **Date**: 2025-11-28 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/1-interactive-raycasting-points/spec.md`

## Summary

建立一個互動式 3D 點雲視覺化範例，展示 Three.js 的 Raycaster 功能。使用者可以用滑鼠與三種不同類型的點雲進行互動，當滑鼠懸停在點上時會顯示視覺回饋（紅色球體標記），同時場景會自動緩慢旋轉以展示 3D 效果。技術方案採用單一 HTML 檔案結構，透過 CDN 載入 Three.js，實現完整的 WebGL 點雲渲染和 Raycasting 互動功能。

## Technical Context

**Language/Version**: JavaScript ES6+ (ES Modules)  
**Primary Dependencies**: Three.js r160+ (via CDN - unpkg/jsDelivr), Stats.js (FPS 監控)  
**Storage**: N/A (純前端展示，無持久化需求)  
**Testing**: 手動瀏覽器測試 (開啟 HTML 檔案驗證功能)  
**Target Platform**: 現代瀏覽器 (Chrome, Firefox, Safari, Edge) - WebGL 2.0 支援  
**Project Type**: Single HTML file (獨立展示範例)  
**Performance Goals**: 30+ FPS, 100ms 內滑鼠回應, 3 秒內完成載入  
**Constraints**: 最小視窗尺寸 320x240, Raycasting 偵測間隔 20ms (throttle)  
**Scale/Scope**: 3 組點雲 × 12,800 點/組 = 38,400 個點, 40 個球體標記物件池

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 原則 | 狀態 | 說明 |
|------|------|------|
| 簡單性 (Simplicity) | ✅ Pass | 單一 HTML 檔案，無複雜建置流程 |
| 獨立性 (Self-contained) | ✅ Pass | 僅依賴 CDN 載入的 Three.js，可獨立執行 |
| 可測試性 (Testable) | ✅ Pass | 開啟瀏覽器即可手動驗證所有功能 |
| 文件完整性 | ✅ Pass | 規格已完成澄清，無模糊需求 |

**Gate Result**: ✅ PASS - 可進入 Phase 0

## Project Structure

### Documentation (this feature)

```text
specs/1-interactive-raycasting-points/
├── spec.md              # 功能規格 (已完成)
├── plan.md              # 本檔案 - 實作計畫
├── research.md          # Phase 0 輸出 - 技術研究
├── data-model.md        # Phase 1 輸出 - 資料模型
├── quickstart.md        # Phase 1 輸出 - 快速開始指南
└── checklists/
    └── requirements.md  # 需求檢查清單 (已完成)
```

### Source Code (repository root)

```text
examples/
└── interactive-raycasting-points/
    └── index.html       # 單一 HTML 檔案 (內嵌 CSS + JS)
```

**Structure Decision**: 採用極簡結構，單一 HTML 檔案包含所有程式碼，與 Three.js 官方範例結構一致，便於學習和分享。

## Complexity Tracking

> 無違規需要記錄 - 本專案採用最簡單的單檔案結構

---

## Phase 0: Research

### 研究任務

1. **Three.js CDN 載入方式** - 確認 import map 配置和 ES Module 載入語法
2. **BufferGeometry 點雲建立** - 研究位置和顏色屬性的 Float32Array 配置
3. **Raycaster Points 偵測** - 確認 threshold 參數和交集檢測 API
4. **Stats.js 整合** - 確認 FPS 監控面板的載入和更新方式
5. **攝影機旋轉矩陣** - 研究 Matrix4.makeRotationY 應用方式

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
