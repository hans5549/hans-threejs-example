# Quickstart: WebGPU Custom Lighting Model

**Feature**: 13-webgpu-lights-custom  
**Date**: 2025-12-01

## Prerequisites

- 支援 WebGPU 的瀏覽器：
  - Chrome 113+ (推薦)
  - Edge 113+
  - Firefox Nightly (需啟用 flag)
- 本地 HTTP 伺服器（ES Modules 需要）

## Quick Start

### 1. 啟動本地伺服器

從專案根目錄執行：

```bash
# 使用 Python
python -m http.server 8080

# 或使用 Node.js
npx serve .

# 或使用 VS Code Live Server 擴充套件
```

### 2. 開啟範例

在瀏覽器中訪問：
```
http://localhost:8080/examples/webgpu-lights-custom/
```

### 3. 預期結果

- 看到黑色背景上的 50 萬粒子點雲
- 三個彩色光源（橙/藍/綠）在空間中移動
- 光源附近的粒子顯示對應顏色
- 滑鼠可旋轉和縮放視角

## File Structure

```
examples/webgpu-lights-custom/
├── index.html          # 主要範例檔案
└── assets/
    └── fallback.png    # WebGPU 不支援時的替代圖片
```

## Interaction Guide

| 操作 | 效果 |
|------|------|
| 滑鼠左鍵拖曳 | 旋轉視角 |
| 滑鼠右鍵拖曳 | 平移視角 |
| 滾輪 | 縮放（0-4 單位） |

## Troubleshooting

### 顯示靜態圖片而非互動場景

**原因**: 瀏覽器不支援 WebGPU

**解決方案**:
1. 更新到最新版 Chrome 或 Edge
2. 在 Chrome 中檢查 `chrome://gpu` 確認 WebGPU 狀態
3. 嘗試重啟瀏覽器

### 黑屏無任何顯示

**可能原因**:
1. JavaScript 錯誤 - 開啟 DevTools Console 檢查
2. Import map 載入失敗 - 檢查網路連線
3. GPU 驅動問題 - 更新顯示驅動程式

### 效能不佳 (< 30 FPS)

**解決方案**:
1. 關閉其他 GPU 密集型應用
2. 檢查是否使用獨立顯卡而非內建 GPU
3. 降低瀏覽器視窗大小

## Development

### 修改粒子數量

在 `index.html` 中找到：
```javascript
for ( let i = 0; i < 500000; i ++ ) {
```
修改 `500000` 為所需數量。

### 修改光源顏色

在 `init()` 函式中找到：
```javascript
light1 = addLight( 0xffaa00 );  // 橙色
light2 = addLight( 0x0040ff );  // 藍色
light3 = addLight( 0x80ff80 );  // 綠色
```
修改十六進位顏色值。

### 修改光源移動速度

在 `animate()` 函式中調整頻率參數：
```javascript
light1.position.x = Math.sin( time * 0.7 ) * scale;  // 0.7 是頻率
```

## Dependencies

此範例透過 CDN 載入 Three.js，無需本地安裝：

- Three.js r170+ (WebGPU build)
- Three.js TSL
- OrbitControls

## Browser Support Matrix

| Browser | Version | Status |
|---------|---------|--------|
| Chrome | 113+ | ✅ 完整支援 |
| Edge | 113+ | ✅ 完整支援 |
| Firefox | Nightly | ⚠️ 需啟用 flag |
| Safari | 18+ | ⚠️ 部分支援 |
| Mobile Chrome | - | ❌ 尚未支援 |
| Mobile Safari | - | ❌ 尚未支援 |
