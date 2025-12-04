# Quickstart: WebGL Geometry Terrain

**Feature**: 016-geometry-terrain
**Date**: 2025-12-04

## Prerequisites

- 現代瀏覽器 (Chrome, Firefox, Safari, Edge)
- 支援 WebGL (大多數現代瀏覽器已支援)
- 本機 HTTP 伺服器 (ES Modules 需要)

## Quick Setup

### 1. 開啟範例

```bash
# 從專案根目錄
cd examples/webgl-geometry-terrain

# 使用任意 HTTP 伺服器
npx serve .
# 或
python -m http.server 8080
```

### 2. 瀏覽器開啟

```text
http://localhost:8080
```

## Controls

| 輸入 | 動作 |
|------|------|
| 滑鼠左鍵 | 向前移動 |
| 滑鼠右鍵 | 向後移動 |
| 滑鼠移動 | 視角旋轉 |
| W / ↑ | 向前移動 |
| S / ↓ | 向後移動 |
| A / ← | 向左移動 |
| D / → | 向右移動 |

## Expected Behavior

1. 頁面載入後顯示程序化生成的 3D 地形
2. 地形具有明暗紋理，呈現光照效果
3. 遠處地形因霧效果逐漸融入暖色調背景
4. 可使用滑鼠和鍵盤自由探索場景
5. 左上角顯示 FPS 效能監控

## Troubleshooting

### 畫面空白或黑屏

- 檢查瀏覽器控制台是否有錯誤
- 確認使用 HTTP 伺服器而非直接開啟檔案
- 確認瀏覽器支援 WebGL

### 效能不佳

- 關閉其他佔用 GPU 的應用程式
- 使用獨立顯卡而非整合顯示晶片
- 確認瀏覽器未在低功耗模式

### WebGL 不支援

- 系統將自動顯示 2D Canvas 降級版本
- 顯示靜態地形圖和提示訊息

## File Structure

```text
examples/webgl-geometry-terrain/
└── index.html    # 完整實作 (單一檔案)
```

## Key Parameters (可調整)

```javascript
// 地形尺寸
const worldWidth = 256;
const worldDepth = 256;

// 物理尺寸
const planeWidth = 7500;
const planeDepth = 7500;

// 高度縮放
const heightScale = 10;

// 霧密度
const fogDensity = 0.0025;

// 控制器速度
controls.movementSpeed = 150;
controls.lookSpeed = 0.1;
```
