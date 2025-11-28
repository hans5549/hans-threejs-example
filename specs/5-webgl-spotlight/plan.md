````markdown
# Implementation Plan: WebGL Spotlight 互動展示

**Branch**: `5-webgl-spotlight` | **Date**: 2025-11-28 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/5-webgl-spotlight/spec.md`

## Summary

建立一個互動式 WebGL Spotlight（聚光燈）展示範例，展示 Three.js 的聚光燈照明、陰影渲染、紋理投射功能。場景包含一個 Lucy 雕像 3D 模型和接收陰影的地面，聚光燈會自動環繞移動產生動態光影效果。使用者可透過 GUI 控制面板即時調整光源屬性（顏色、強度、角度、衰減）、切換紋理投射、調整陰影參數，並可開啟輔助視覺化工具理解聚光燈工作原理。技術方案採用單一 HTML 檔案結構，透過 CDN 載入 Three.js 及相關資源。

## Technical Context

**Language/Version**: JavaScript ES6+ (ES Modules)  
**Primary Dependencies**: Three.js r160+ (via CDN), lil-gui (GUI 控制面板), PLYLoader (3D 模型載入), OrbitControls (攝影機控制)  
**Storage**: N/A (純前端展示，無持久化需求)  
**Testing**: 手動瀏覽器測試 (開啟 HTML 檔案驗證功能)  
**Target Platform**: 桌面瀏覽器 (Chrome, Firefox, Safari, Edge) - WebGL 2.0 支援  
**Project Type**: Single HTML file (獨立展示範例)  
**Performance Goals**: 60 FPS, 100ms 內 GUI 參數響應, 3 秒內完成載入  
**Constraints**: 僅支援桌面滑鼠操作，攝影機距離 2-10，不低於地平面  
**Scale/Scope**: 單一聚光燈、1 個 3D 模型 (Lucy ~100k 頂點)、3 種紋理貼圖、10 項 GUI 可調參數

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 原則 | 狀態 | 說明 |
|------|------|------|
| 簡單性 (Simplicity) | ✅ Pass | 單一 HTML 檔案，無複雜建置流程 |
| 獨立性 (Self-contained) | ⚠️ Partial | 依賴 Three.js CDN 及外部 3D 模型、紋理資源 |
| 可測試性 (Testable) | ✅ Pass | 開啟瀏覽器即可手動驗證所有功能 |
| 文件完整性 | ✅ Pass | 規格已完成 5 項釐清，無模糊需求 |

**Gate Result**: ✅ PASS - 外部依賴已在規格中明確記錄，具有後備方案

## Project Structure

### Documentation (this feature)

```text
specs/5-webgl-spotlight/
├── spec.md              # 功能規格 (已完成)
├── plan.md              # 本檔案 - 實作計畫
├── research.md          # Phase 0 輸出 - 技術研究
├── data-model.md        # Phase 1 輸出 - 資料模型
├── quickstart.md        # Phase 1 輸出 - 快速開始指南
├── contracts/           # Phase 1 輸出 - 元件介面
│   └── component-interface.md
├── checklists/
│   └── requirements.md  # 需求檢查清單 (已完成)
└── tasks.md             # Phase 2 輸出 (由 /speckit.tasks 產生)
```

### Source Code (repository root)

```text
examples/
└── webgl-spotlight/
    ├── index.html           # 主要 HTML 檔案 (內嵌 CSS + JS)
    └── fallback.png         # WebGL 不支援時的降級靜態圖片
```

**Structure Decision**: 採用與其他範例一致的單一 HTML 檔案結構，僅額外包含 WebGL 降級用的靜態圖片。所有外部資源（3D 模型、紋理）從 Three.js CDN 載入。

## Complexity Tracking

> 無違規需要記錄 - 本專案採用最簡單的單檔案結構

---

## Phase 0: Research

### 研究任務

1. **SpotLight 光源設定** - 研究 Three.js SpotLight 的所有屬性和陰影配置
2. **紋理投射功能** - 確認 SpotLight.map 屬性的使用方式和紋理格式要求
3. **PCF Soft Shadow** - 研究軟陰影渲染的配置和效能考量
4. **PLYLoader 載入 3D 模型** - 確認 Lucy 模型的載入方式和後備處理
5. **lil-gui 控制面板** - 研究 GUI 控制項的建立方式和事件綁定
6. **SpotLightHelper 和 CameraHelper** - 確認輔助視覺化工具的使用方式
7. **Neutral Tone Mapping** - 研究色調映射的配置
8. **WebGL 偵測和降級** - 確認 WebGL 支援偵測和降級展示方式

### 研究結果

詳見 [research.md](./research.md)

---

## Phase 1: Design

### 資料模型

詳見 [data-model.md](./data-model.md)

### 元件介面

詳見 [contracts/component-interface.md](./contracts/component-interface.md)

### 快速開始指南

詳見 [quickstart.md](./quickstart.md)

---

## Phase 2: Tasks

> 由 `/speckit.tasks` 命令產生，詳見 [tasks.md](./tasks.md)

````
