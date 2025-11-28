# Implementation Plan: Unreal Bloom Post-processing Effect

**Branch**: `2-unreal-bloom` | **Date**: 2025-11-28 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/2-unreal-bloom/spec.md`

## Summary

實作 Three.js Unreal Bloom 後處理效果展示應用。核心功能包括：載入動畫 3D 模型（Primary Ion Drive）、應用 UnrealBloomPass 後處理效果、提供 OrbitControls 互動控制、以及 lil-gui 參數調整介面。技術方案參考 Three.js 官方範例 `webgl_postprocessing_unreal_bloom`，採用 ES Modules 架構。

## Technical Context

**Language/Version**: JavaScript ES2022+ (ES Modules)  
**Primary Dependencies**: 
- Three.js (latest via CDN importmap)
- lil-gui (GUI 控制面板)
- stats.js (效能監控)

**Storage**: N/A (純前端應用，無持久化需求)  
**Testing**: 手動視覺測試 + 瀏覽器開發者工具效能分析  
**Target Platform**: 現代瀏覽器（Chrome、Firefox、Safari、Edge 最新兩個主要版本）
**Project Type**: single/web - 單一 HTML 檔案應用  
**Performance Goals**: 維持 30+ FPS，GUI 參數調整響應 <100ms  
**Constraints**: 
- 3D 模型從 Three.js 官方 CDN 載入
- WebGL 不支援時提供 2D 降級版本
- 模型載入失敗時提供重試機制

**Scale/Scope**: 單頁展示應用，無複雜狀態管理需求

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| 程式碼品質 | ✅ PASS | 使用官方推薦的 ES Modules 架構 |
| 可維護性 | ✅ PASS | 單檔案結構，程式碼組織清晰 |
| 效能標準 | ✅ PASS | 目標 30+ FPS，符合 Three.js 範例標準 |
| 錯誤處理 | ✅ PASS | 規劃模型載入失敗重試、WebGL 降級機制 |
| 瀏覽器相容性 | ✅ PASS | 限定現代瀏覽器，避免相容性問題 |

**Gate Status**: ✅ PASSED - 可進入 Phase 0 研究階段

## Project Structure

### Documentation (this feature)

```text
specs/2-unreal-bloom/
├── spec.md              # 功能規格（已完成）
├── plan.md              # 本檔案 - 實作計畫
├── research.md          # Phase 0 輸出 - 技術研究
├── data-model.md        # Phase 1 輸出 - 資料模型
├── quickstart.md        # Phase 1 輸出 - 快速開始指南
├── contracts/           # Phase 1 輸出 - API 契約（本功能為前端，不適用）
├── tasks.md             # Phase 2 輸出 - 任務清單
└── checklists/
    └── requirements.md  # 需求檢查清單（已完成）
```

### Source Code (repository root)

```text
examples/
└── unreal-bloom-postprocessing/
    └── index.html       # 主要應用程式檔案（單檔案架構）
```

**Structure Decision**: 採用單一 HTML 檔案架構，與 Three.js 官方範例保持一致。所有 JavaScript 程式碼以 `<script type="module">` 內嵌於 HTML 中，便於部署和分享。

## Complexity Tracking

> 無違反項目需要記錄

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| N/A | N/A | N/A |

## Phase 0: Research Summary

詳見 [research.md](./research.md)

### Key Decisions Made

1. **渲染管線**: 使用 EffectComposer + RenderPass + UnrealBloomPass + OutputPass
2. **模型格式**: GLTF/GLB 格式，使用 GLTFLoader 載入
3. **動畫系統**: 使用 AnimationMixer 播放模型內建動畫
4. **GUI 函式庫**: 使用 lil-gui（Three.js 官方範例標準）
5. **相機控制**: 使用 OrbitControls 提供軌道控制
6. **CDN 策略**: 使用 importmap 從 Three.js CDN 載入所有依賴

## Phase 1: Design Summary

詳見 [data-model.md](./data-model.md) 和 [quickstart.md](./quickstart.md)

### Core Components

| 元件 | 職責 | 依賴 |
|------|------|------|
| Scene | 3D 場景容器 | Three.js core |
| PerspectiveCamera | 透視相機 | Three.js core |
| WebGLRenderer | WebGL 渲染器 | Three.js core |
| EffectComposer | 後處理管線 | postprocessing addon |
| UnrealBloomPass | Bloom 效果 | postprocessing addon |
| GLTFLoader | 模型載入 | loaders addon |
| AnimationMixer | 動畫播放 | Three.js core |
| OrbitControls | 相機控制 | controls addon |
| GUI | 參數介面 | lil-gui |
| Stats | 效能監控 | stats.js |

### Parameter Configuration

| 參數 | 預設值 | 範圍 | 用途 |
|------|--------|------|------|
| threshold | 0 | 0.0 - 1.0 | Bloom 亮度閾值 |
| strength | 1 | 0.0 - 3.0 | Bloom 強度 |
| radius | 0.5 | 0.0 - 1.0 | Bloom 擴散半徑 |
| exposure | 1 | 0.1 - 2.0 | 色調映射曝光度 |

## Next Steps

執行 `/speckit.tasks` 產生詳細任務清單後，即可開始實作。
