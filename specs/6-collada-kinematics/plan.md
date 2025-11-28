# Implementation Plan: COLLADA Kinematics 機器人手臂動畫

**Branch**: `6-collada-kinematics` | **Date**: 2025-11-28 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/6-collada-kinematics/spec.md`

## Summary

建立互動式 3D 工業機器人手臂展示，使用 Three.js ColladaLoader 載入 COLLADA 格式的機器人模型，透過 TWEEN.js 控制關節運動學動畫。支援 3 種機器人模型切換（ABB IRB 52、KUKA KR5、Universal Robots UR6），提供即時 FPS 監控、自動環繞攝影機和載入進度顯示。

## Technical Context

**Language/Version**: JavaScript (ES2020+)  
**Primary Dependencies**: 
- Three.js r160+ (3D 渲染引擎)
- ColladaLoader (COLLADA 模型載入器)
- TWEEN.js (動畫補間引擎)
- Stats.js (效能監控)

**Storage**: N/A (靜態網頁應用)  
**Testing**: 手動測試 + 瀏覽器開發工具  
**Target Platform**: 現代瀏覽器 (Chrome, Firefox, Safari, Edge) + WebGL 支援  
**Project Type**: Single HTML + Static Assets  
**Performance Goals**: 30+ FPS, 模型載入 < 5 秒  
**Constraints**: WebGL 必要、模型檔案預先託管  
**Scale/Scope**: 單一展示頁面、3 種機器人模型

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Gate | Status | Notes |
|------|--------|-------|
| 專案結構符合現有模式 | ✅ PASS | 遵循 `examples/[feature-name]/index.html` 結構 |
| 相依套件最小化 | ✅ PASS | 僅使用 Three.js 官方 addons |
| 無需後端服務 | ✅ PASS | 純前端靜態應用 |
| 模型檔案可取得 | ✅ PASS | ABB 從 Three.js；KUKA/UR 從 collada_robots |

**Post-Phase 1 Re-Check**: ✅ 所有 gates 通過

## Project Structure

### Documentation (this feature)

```text
specs/6-collada-kinematics/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output
│   └── component-interface.md
├── checklists/
│   └── requirements.md  # Spec quality checklist
└── tasks.md             # Phase 2 output (/speckit.tasks)
```

### Source Code (repository root)

```text
examples/
└── collada-kinematics/
    ├── index.html       # 主要應用程式（單一 HTML 檔案）
    ├── fallback.svg     # WebGL 不支援時的降級圖片
    └── models/          # 機器人模型檔案目錄
        ├── abb_irb52_7_120.dae
        ├── kuka-kr5-r650.dae
        └── universalrobots-ur6-85-5-a.dae
```

**Structure Decision**: 採用單一 HTML 檔案結構，與現有專案範例（webgl-spotlight, css3d-periodic-table 等）保持一致。模型檔案統一放置於 `models/` 子目錄。

## Complexity Tracking

> 無違規項目需要記錄

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| - | - | - |

## Phase 0: Research Tasks ✅ COMPLETED

1. ✅ 確認 ABB IRB 52 模型來源和格式（Three.js 官方範例已提供）
2. ✅ 研究 KUKA KR5-R650 模型取得方式（collada_robots 倉庫 .zae）
3. ✅ 研究 Universal Robots UR6 模型取得方式（collada_robots 倉庫 .zae）
4. ✅ 確認 .zae 解壓縮流程（ZIP 格式，使用 unzip）
5. ✅ 確認 ColladaLoader 對不同模型的相容性（支援，需測試 kinematics）

**Output**: [research.md](./research.md)

## Phase 1: Design Tasks ✅ COMPLETED

1. ✅ 定義機器人模型資料結構 (Robot Catalog) → [data-model.md](./data-model.md)
2. ✅ 設計 UI 元件介面（選擇器、載入指示器）→ [contracts/component-interface.md](./contracts/component-interface.md)
3. ✅ 設計場景管理流程（模型切換、動畫重置）→ [contracts/component-interface.md](./contracts/component-interface.md)
4. ✅ 建立 quickstart 指南 → [quickstart.md](./quickstart.md)

**Output**: data-model.md, contracts/component-interface.md, quickstart.md

## Dependencies

| 相依項 | 版本 | 用途 |
|--------|------|------|
| three | r160+ | 3D 渲染引擎 |
| ColladaLoader | (three/addons) | COLLADA 模型載入 |
| TWEEN | (three/addons/libs) | 動畫補間 |
| Stats | (three/addons/libs) | FPS 監控 |

## Risk Assessment

| 風險 | 影響 | 緩解策略 |
|------|------|----------|
| KUKA/UR 模型無法取得或格式不相容 | 高 | 使用 ABB 單一模型作為降級方案 |
| 模型檔案過大影響載入時間 | 中 | 顯示載入進度、考慮模型壓縮 |
| 部分模型缺少運動學資訊 | 中 | 顯示靜態模型並提示使用者 |
