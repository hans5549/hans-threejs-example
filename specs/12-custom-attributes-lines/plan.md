# Implementation Plan: WebGL Custom Attributes Lines

**Branch**: `12-custom-attributes-lines` | **Date**: 2025-12-01 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/12-custom-attributes-lines/spec.md`

## Summary

實作一個使用 Three.js ShaderMaterial 和自訂頂點屬性的 3D 文字線條動畫展示。系統將載入 Helvetiker Bold 字型建立「three.js」文字的 TextGeometry，透過自訂 vertex shader 實現動態位移效果，並使用 fragment shader 配合 HSL 顏色漸變。採用 Additive Blending 渲染半透明線條，並實現自動旋轉動畫。字型載入失敗時使用備用 BoxGeometry 替代。

## Technical Context

**Language/Version**: JavaScript ES6+ Modules  
**Primary Dependencies**: Three.js r174+, FontLoader, TextGeometry, Stats.js  
**Storage**: N/A（純前端應用）  
**Testing**: 手動瀏覽器測試  
**Target Platform**: 現代瀏覽器（Chrome, Firefox, Safari, Edge），需支援 WebGL  
**Project Type**: Single HTML file（遵循專案現有結構）  
**Performance Goals**: ≥30 FPS，頁面載入 ≤3 秒  
**Constraints**: 固定文字「three.js」，相機位置 Z=400  
**Scale/Scope**: 單一展示頁面

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 原則 | 狀態 | 說明 |
|------|------|------|
| 結構一致性 | ✅ Pass | 遵循現有 `examples/` 目錄結構 |
| 相依性管理 | ✅ Pass | 使用 CDN import map，與現有範例一致 |
| 效能要求 | ✅ Pass | 已定義明確的 FPS 目標 (≥30 FPS) |
| 無額外複雜度 | ✅ Pass | 單一 HTML 檔案，無額外建置流程 |

## Project Structure

### Documentation (this feature)

```text
specs/12-custom-attributes-lines/
├── spec.md              # 功能規格
├── plan.md              # 本檔案
├── research.md          # Phase 0 研究輸出
├── data-model.md        # Phase 1 資料模型
├── quickstart.md        # Phase 1 快速入門指南
├── contracts/           # Phase 1 元件介面
│   └── component-interface.md
├── tasks.md             # Phase 2 任務清單（由 /speckit.tasks 產生）
└── checklists/
    └── requirements.md  # 規格驗證檢查清單
```

### Source Code (repository root)

```text
examples/
└── webgl-custom-attributes-lines/
    ├── index.html       # 主要展示頁面（含所有 JavaScript 和 GLSL shaders）
    └── fallback.png     # WebGL 不支援時的替代圖片
```

**Structure Decision**: 使用單一 HTML 檔案結構，與專案中其他範例保持一致。所有 JavaScript 程式碼和 GLSL shader 程式碼內嵌於 HTML 中，透過 CDN import map 載入 Three.js 相依套件。

## Complexity Tracking

> **無違規需要記錄** - 本功能完全符合專案結構規範。

---

## Phase 0: 研究與調查

### 待研究項目

1. **ShaderMaterial 與自訂 Attributes 機制**
   - BufferAttribute 如何與 vertex shader 中的 attribute 變數對應
   - uniform 變數的定義與動態更新
   - varying 變數在 vertex 和 fragment shader 間的傳遞

2. **TextGeometry 與 FontLoader**
   - 字型檔案格式 (typeface.json)
   - TextGeometry 參數配置 (size, depth, curveSegments, bevel)
   - 頂點數量與 attribute 陣列大小的關係

3. **Additive Blending 渲染**
   - THREE.AdditiveBlending 與其他混合模式的差異
   - depthTest: false 對渲染順序的影響
   - transparent: true 的必要性

4. **動畫迴圈與效能**
   - requestAnimationFrame 與 setAnimationLoop 的使用
   - attribute.needsUpdate 標記機制
   - uniform 更新的效能考量

5. **字型載入失敗處理**
   - FontLoader 錯誤回調機制
   - BoxGeometry 作為備用幾何體的實作

### 研究輸出

→ 請參閱 [research.md](research.md)

---

## Phase 1: 設計與合約

### 資料模型設計

→ 請參閱 [data-model.md](data-model.md)

### 元件介面合約

→ 請參閱 [contracts/component-interface.md](contracts/component-interface.md)

### 快速入門指南

→ 請參閱 [quickstart.md](quickstart.md)

---

## Phase 2: 任務分解

→ 由 `/speckit.tasks` 命令產生，請參閱 [tasks.md](tasks.md)
