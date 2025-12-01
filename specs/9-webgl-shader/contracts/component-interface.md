# Component Interface: WebGL Shader

**Feature**: 9-webgl-shader  
**Date**: 2025-12-01  
**Status**: Complete

## 概述

定義 WebGL Shader 範例的元件介面和程式碼結構。

---

## 檔案結構

```
examples/webgl-shader/
└── index.html          # 主要展示頁面
```

---

## HTML 結構

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <title>three.js webgl - shader [Monjori]</title>
    <meta charset="utf-8">
    <meta name="viewport" content="...">
    <style>/* 內嵌樣式 */</style>
</head>
<body>
    <div id="container"></div>
    <div id="info"><!-- 資訊和連結 --></div>
    
    <script id="vertexShader" type="x-shader/x-vertex">
        // GLSL Vertex Shader
    </script>
    
    <script id="fragmentShader" type="x-shader/x-fragment">
        // GLSL Fragment Shader (Monjori)
    </script>
    
    <script type="importmap">
        // ES Modules import map
    </script>
    
    <script type="module">
        // 主要 JavaScript 程式碼
    </script>
</body>
</html>
```

---

## JavaScript API

### 全域函式

#### `init()`

初始化 Three.js 場景和渲染環境。

```typescript
function init(): void
```

**職責**:
- 取得 container DOM 元素
- 建立 OrthographicCamera
- 建立 Scene
- 建立 PlaneGeometry
- 建立 uniforms 物件
- 建立 ShaderMaterial
- 建立 Mesh 並加入 Scene
- 建立 WebGLRenderer
- 設定 Animation Loop
- 註冊 resize 事件監聽器

---

#### `onWindowResize()`

處理視窗大小調整事件。

```typescript
function onWindowResize(): void
```

**職責**:
- 更新 renderer 尺寸為新的視窗大小

---

#### `animate()`

動畫迴圈回呼函式。

```typescript
function animate(): void
```

**職責**:
- 更新 `uniforms.time.value` 為當前時間
- 呼叫 `renderer.render(scene, camera)`

---

## GLSL Shader 介面

### Vertex Shader

```glsl
// 輸入 (由 Three.js 提供)
attribute vec3 position;    // 頂點位置
attribute vec2 uv;          // 紋理座標

// 輸出 (傳遞給 Fragment Shader)
varying vec2 vUv;           // UV 座標

void main() {
    vUv = uv;
    gl_Position = vec4(position, 1.0);
}
```

### Fragment Shader

```glsl
// 輸入
varying vec2 vUv;           // 從 Vertex Shader 接收的 UV
uniform float time;         // 動畫時間 (秒)

// 輸出
// gl_FragColor: vec4       // 像素顏色 (RGBA)

void main() {
    // Monjori 演算法
    vec2 p = -1.0 + 2.0 * vUv;
    // ... 計算顏色 ...
    gl_FragColor = vec4(color, 1.0);
}
```

---

## CSS 樣式介面

### 必要樣式

```css
body {
    margin: 0;              /* 移除預設邊距 */
    overflow: hidden;       /* 隱藏捲軸 */
}

#info {
    position: absolute;     /* 絕對定位 */
    top: 10px;              /* 頂部對齊 */
    width: 100%;            /* 全寬 */
    text-align: center;     /* 文字置中 */
    color: #fff;            /* 白色文字 */
    z-index: 100;           /* 確保在 canvas 上方 */
}

#info a {
    color: #4af;            /* 連結顏色 */
}
```

---

## Import Map 配置

```json
{
    "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
    }
}
```

---

## 事件介面

| 事件 | 處理函式 | 說明 |
|------|----------|------|
| `resize` | `onWindowResize()` | 視窗大小改變 |
| Animation Frame | `animate()` | 每幀更新 (由 setAnimationLoop 管理) |

---

## 驗證介面

### 視覺驗證

- [ ] 頁面載入後立即顯示動態效果
- [ ] 效果持續動態變化
- [ ] 調整視窗後效果正常填滿

### 控制台驗證

- [ ] 無 JavaScript 錯誤
- [ ] 無 WebGL 錯誤
- [ ] 無 Shader 編譯錯誤

### 連結驗證

- [ ] Three.js 連結可點擊並開啟新分頁
- [ ] Monjori 原作連結可點擊並開啟新分頁
