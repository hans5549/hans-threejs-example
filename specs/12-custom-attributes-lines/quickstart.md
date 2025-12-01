# Quick Start Guide: WebGL Custom Attributes Lines

**Feature**: 12-custom-attributes-lines  
**Date**: 2025-12-01

## 概述

本功能實作 Three.js WebGL 自訂屬性線條範例，展示如何使用 ShaderMaterial 搭配自訂 buffer attributes 建立動態的 3D 線條文字動畫。

## 先決條件

- 現代瀏覽器（Chrome 90+、Firefox 88+、Safari 14+、Edge 90+）
- 支援 WebGL 的顯示卡
- 網路連線（CDN 載入依賴）

## 快速開始

### 1. 檔案結構

```
examples/
└── webgl-custom-attributes-lines/
    └── index.html
```

### 2. 執行範例

```bash
# 使用 Python HTTP Server
cd examples/webgl-custom-attributes-lines
python -m http.server 8080

# 或使用 Node.js serve
npx serve .

# 開啟瀏覽器
open http://localhost:8080
```

### 3. 預期結果

- 畫面顯示 "three.js" 3D 文字
- 文字以線條模式渲染
- 線條持續產生位移動畫效果
- 顏色隨時間漸變（HSL 偏移）
- 左上角顯示 FPS 統計

## 核心概念

### 自訂 Buffer Attributes

本範例使用兩個自訂 buffer attributes：

1. **displacement** - 控制頂點位移量
2. **customColor** - 控制每個頂點的顏色

### Shader Uniforms

動態參數透過 uniforms 傳遞至 shader：

- `amplitude` - 位移振幅（sin 波動）
- `opacity` - 透明度
- `color` - 基礎顏色（與 vColor 相乘）

### 加法混色

使用 `THREE.AdditiveBlending` 產生發光效果，搭配 `depthTest: false` 確保重疊線條正確顯示。

## 自訂指南

### 修改文字內容

```javascript
const geometry = new TextGeometry('YOUR TEXT', {
  font: font,
  size: 50,
  depth: 15,
  // ...其他參數
});
```

### 調整動畫速度

```javascript
// 在 render() 函式中
const time = Date.now() * 0.001; // 基準時間
line.rotation.y = 0.25 * time;   // 調整旋轉速度
uniforms.amplitude.value = Math.sin(0.5 * time); // 調整波動頻率
```

### 修改顏色效果

```javascript
// 初始顏色
uniforms.color.value = new THREE.Color(0xff0000); // 紅色

// HSL 偏移速度
uniforms.color.value.offsetHSL(0.001, 0, 0); // 加快色相變化
```

### 調整位移效果

```javascript
// 在 render() 函式中
displacement.array[i] += (Math.random() - 0.5) * delta;
```

## 故障排除

### 畫面空白

1. 檢查瀏覽器控制台錯誤訊息
2. 確認 WebGL 支援：`chrome://gpu`
3. 確認 CDN 連線正常

### 字型載入失敗

- 範例會自動切換至 BoxGeometry 備用方案
- 檢查網路連線
- 確認 CDN URL 正確

### 效能不佳

- 減少 TextGeometry 的 `curveSegments` 和 `bevelSegments`
- 降低 displacement 更新頻率
- 使用 `renderer.setPixelRatio(1)` 降低解析度

## 延伸閱讀

- [Three.js ShaderMaterial 文件](https://threejs.org/docs/#api/en/materials/ShaderMaterial)
- [Three.js BufferAttribute 文件](https://threejs.org/docs/#api/en/core/BufferAttribute)
- [GLSL Shaders 教學](https://thebookofshaders.com/)
- [原始範例](https://threejs.org/examples/#webgl_custom_attributes_lines)
