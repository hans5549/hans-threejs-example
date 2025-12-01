# Implementation Plan: Particle Earth Sphere

**Branch**: `14-particle-earth` | **Date**: 2025-12-01 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/14-particle-earth/spec.md`

## Summary

建立一個由 50,000 個粒子組成的地球球體視覺化效果。粒子使用斐波那契球面分布演算法均勻分布於球體表面，顏色從 NASA Blue Marble 地球紋理貼圖取樣。系統支援滑鼠互動控制攝影機視角，並提供 GUI 控制面板調整粒子大小和旋轉速度。

## Technical Context

**Language/Version**: JavaScript ES6+, HTML5, GLSL ES 3.0  
**Primary Dependencies**: Three.js r160+, lil-gui  
**Storage**: N/A（純前端應用）  
**Testing**: 手動視覺測試 + 瀏覽器 DevTools 效能分析  
**Target Platform**: 現代瀏覽器（Chrome, Firefox, Safari, Edge）支援 WebGL  
**Project Type**: Single HTML Web Application  
**Performance Goals**: 60 FPS, 3 秒內完成初始渲染  
**Constraints**: <16ms 每幀渲染時間, 50,000 粒子  
**Scale/Scope**: 單頁展示應用

## Constitution Check

*GATE: 此專案為範例展示，遵循現有專案結構模式*

| 檢查項目 | 狀態 | 說明 |
|---------|------|------|
| 單一 HTML 檔案結構 | ✅ Pass | 遵循 `examples/` 目錄下的現有模式 |
| Three.js CDN 引入 | ✅ Pass | 使用 unpkg CDN，與現有範例一致 |
| 無外部建構工具依賴 | ✅ Pass | 純 ES Module，無需打包 |
| 自包含範例 | ✅ Pass | 所有程式碼在單一檔案中 |

## Project Structure

### Documentation (this feature)

```text
specs/14-particle-earth/
├── plan.md              # This file
├── research.md          # Phase 0 output - 技術研究
├── data-model.md        # Phase 1 output - 資料模型
├── quickstart.md        # Phase 1 output - 快速入門
├── contracts/           # Phase 1 output - API 契約
│   └── shader-uniforms.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit.tasks)
```

### Source Code (repository root)

```text
examples/
└── particle-earth-sphere/
    ├── index.html           # 主要應用程式（單一檔案）
    └── assets/
        ├── earth-map.jpg    # NASA Blue Marble 地球紋理
        └── earth-fallback.jpg  # WebGL 不支援時的降級圖片
```

**Structure Decision**: 採用單一 HTML 檔案結構，遵循專案中其他 Three.js 範例的模式。所有 JavaScript 和 GLSL Shader 程式碼內嵌於 HTML 中，使用 ES Module 從 CDN 載入 Three.js。

## Technical Approach

### Core Algorithm: 斐波那契球面分布

使用黃金角演算法在球面上均勻分布 50,000 個粒子：

```
φ = π * (3 - √5)  // 黃金角
for i = 0 to n-1:
    y = 1 - (i / (n-1)) * 2  // y 從 1 到 -1
    radius = √(1 - y²)
    θ = φ * i
    x = cos(θ) * radius
    z = sin(θ) * radius
```

### Shader Strategy

**Vertex Shader**:
- 接收球面座標位置
- 套用地球旋轉變換
- 計算 UV 座標用於紋理取樣
- 設定粒子大小

**Fragment Shader**:
- 從地球紋理取樣顏色
- 套用加法混合產生發光效果
- 處理透明度

### Camera Interaction

- 滑鼠位置映射到攝影機位置偏移
- 使用線性插值（lerp）實現平滑跟隨
- 滑鼠離開時緩慢回歸預設位置
- 攝影機始終朝向球心

## Dependencies

| 套件 | 版本 | 用途 |
|------|------|------|
| Three.js | r160+ | 3D 渲染引擎 |
| lil-gui | latest | GUI 控制面板 |

## Risk Assessment

| 風險 | 影響 | 緩解措施 |
|------|------|----------|
| 50,000 粒子效能問題 | 低 FPS | 使用 BufferGeometry 最佳化記憶體配置 |
| 紋理載入失敗 | 無法顯示顏色 | 提供降級的純色粒子模式 |
| WebGL 不支援 | 無法渲染 | 顯示靜態圖片降級方案 |
| 攝影機插值抖動 | 視覺不佳 | 調整插值因子，使用 requestAnimationFrame |

## Complexity Tracking

> 無違反項目，此功能遵循既有專案模式。

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| N/A | - | - |
