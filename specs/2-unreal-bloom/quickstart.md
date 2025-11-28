# Quickstart: Unreal Bloom Post-processing Effect

**Feature**: 2-unreal-bloom  
**Date**: 2025-11-28

## Overview

Three.js Unreal Bloom 後處理效果展示應用。載入動畫 3D 模型並套用高品質 Bloom 發光效果，提供即時參數調整介面。

## Prerequisites

- 現代瀏覽器（Chrome、Firefox、Safari、Edge 最新版本）
- 支援 WebGL 的裝置
- 網路連線（載入 CDN 資源和 3D 模型）

## Quick Start

### 1. 開啟應用

使用本地伺服器開啟 `examples/unreal-bloom-postprocessing/index.html`：

```bash
# 使用 Python
cd examples/unreal-bloom-postprocessing
python -m http.server 8080

# 或使用 Node.js
npx serve .
```

然後在瀏覽器開啟 `http://localhost:8080`

### 2. 基本操作

| 操作 | 效果 |
|------|------|
| 滑鼠左鍵拖曳 | 旋轉視角 |
| 滑鼠滾輪 | 縮放 |
| GUI 滑桿 | 調整 Bloom 效果 |

### 3. Bloom 參數說明

| 參數 | 範圍 | 預設值 | 說明 |
|------|------|--------|------|
| threshold | 0 - 1 | 0 | 控制哪些亮度以上的像素會產生 Bloom |
| strength | 0 - 3 | 1 | Bloom 效果的整體強度 |
| radius | 0 - 1 | 0.5 | Bloom 光暈的擴散範圍 |
| exposure | 0.1 - 2 | 1 | 整體畫面曝光度 |

## Project Structure

```
examples/unreal-bloom-postprocessing/
└── index.html    # 單一應用程式檔案
```

## Dependencies (via CDN)

- Three.js (core)
- Three.js addons:
  - EffectComposer
  - RenderPass
  - UnrealBloomPass
  - OutputPass
  - GLTFLoader
  - OrbitControls
  - lil-gui
  - Stats

## Architecture

```
index.html
├── HTML Structure
│   ├── <div id="container">  # 3D 場景容器
│   └── <div id="info">       # 說明文字
│
├── Import Map
│   └── three + addons from CDN
│
└── JavaScript Module
    ├── init()                # 初始化場景、相機、燈光
    ├── loadModel()           # 載入 GLTF 模型
    ├── setupPostProcessing() # 設定後處理管線
    ├── setupGUI()            # 建立參數控制介面
    ├── setupControls()       # 設定相機控制
    ├── onWindowResize()      # 視窗調整處理
    └── animate()             # 動畫迴圈
```

## Key Code Patterns

### 後處理管線設定

```javascript
// 建立效果合成器
const composer = new EffectComposer(renderer);

// 新增 Pass
composer.addPass(new RenderPass(scene, camera));
composer.addPass(new UnrealBloomPass(resolution, strength, radius, threshold));
composer.addPass(new OutputPass());

// 在動畫迴圈中使用 composer.render() 取代 renderer.render()
```

### GUI 參數綁定

```javascript
const params = { threshold: 0, strength: 1, radius: 0.5, exposure: 1 };

gui.add(params, 'threshold', 0, 1).onChange(value => {
  bloomPass.threshold = Number(value);
});
```

## Troubleshooting

### 畫面全黑

- 檢查瀏覽器是否支援 WebGL
- 確認模型已成功載入（開發者工具 Network 面板）
- 確認 CDN 資源可存取

### Bloom 效果不明顯

- 調高 strength 參數
- 調低 threshold 參數
- 確認場景中有足夠亮的區域

### 效能不佳

- 降低瀏覽器視窗大小
- 確認沒有其他高負載應用程式執行中
- 檢查 GPU 驅動程式是否為最新版本

## References

- [Three.js 官方範例](https://threejs.org/examples/#webgl_postprocessing_unreal_bloom)
- [UnrealBloomPass 文件](https://threejs.org/docs/#examples/en/postprocessing/UnrealBloomPass)
- [EffectComposer 文件](https://threejs.org/docs/#examples/en/postprocessing/EffectComposer)
