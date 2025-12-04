# API Contracts: WebGL Effects ASCII

**Feature**: 017-effects-ascii  
**Date**: 2025-01-27  
**Status**: Complete

## Overview

本範例為純前端展示應用，無後端 API。此文件定義內部模組和函式的介面契約。

---

## Module Contracts

### 1. Initialization Contract

```typescript
/**
 * 初始化整個應用程式
 * @returns void
 * @sideEffects 
 *   - 建立 scene, camera, renderer, effect, controls
 *   - 將 effect.domElement 添加到 document.body
 *   - 啟動動畫迴圈
 */
function init(): void;
```

### 2. Scene Setup Contract

```typescript
/**
 * 建立並配置 Three.js 場景
 * @returns THREE.Scene - 已配置的場景物件
 * @sideEffects 
 *   - 建立 sphere 和 plane geometry
 *   - 建立並添加 2 個 PointLight
 */
function createScene(): THREE.Scene;
```

### 3. Camera Setup Contract

```typescript
/**
 * 建立透視相機
 * @returns THREE.PerspectiveCamera - 已定位的相機
 * @postcondition camera.position = (0, 150, 500)
 */
function createCamera(): THREE.PerspectiveCamera;
```

### 4. Renderer Setup Contract

```typescript
/**
 * 建立 WebGL 渲染器
 * @returns THREE.WebGLRenderer - 全螢幕尺寸的渲染器
 * @postcondition renderer.domElement 尚未添加到 DOM
 */
function createRenderer(): THREE.WebGLRenderer;
```

### 5. ASCII Effect Setup Contract

```typescript
/**
 * 建立 ASCII 效果器
 * @param renderer - WebGL 渲染器實例
 * @returns AsciiEffect - 已配置的效果器
 * @postcondition 
 *   - charSet = ' .:-+*=%@#'
 *   - options.invert = true
 *   - domElement.style.color = 'white'
 *   - domElement.style.backgroundColor = 'black'
 */
function createAsciiEffect(renderer: THREE.WebGLRenderer): AsciiEffect;
```

### 6. Controls Setup Contract

```typescript
/**
 * 建立軌跡球控制器
 * @param camera - 要控制的相機
 * @param domElement - 接收事件的 DOM 元素
 * @returns TrackballControls - 已綁定的控制器
 * @precondition domElement 必須已添加到 DOM
 */
function createControls(
    camera: THREE.PerspectiveCamera, 
    domElement: HTMLElement
): TrackballControls;
```

### 7. Animation Contract

```typescript
/**
 * 動畫迴圈函式
 * @sideEffects
 *   - 更新 sphere 位置 (y = abs(sin(time * 2)) * 150)
 *   - 更新 sphere 旋轉 (x += time * 0.0003, z += time * 0.0002)
 *   - 呼叫 controls.update()
 *   - 呼叫 effect.render(scene, camera)
 * @recursive 使用 requestAnimationFrame 自我呼叫
 */
function animate(): void;
```

### 8. Resize Handler Contract

```typescript
/**
 * 視窗大小改變處理函式
 * @sideEffects
 *   - 更新 camera.aspect
 *   - 呼叫 camera.updateProjectionMatrix()
 *   - 呼叫 renderer.setSize()
 *   - 呼叫 effect.setSize()
 * @eventBinding window.resize
 */
function onWindowResize(): void;
```

---

## DOM Contract

### HTML Structure

```html
<!DOCTYPE html>
<html>
<head>
    <title>three.js webgl - effects - ascii</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, user-scalable=no, minimum-scale=1.0, maximum-scale=1.0">
    <style>
        /* 全螢幕無邊距樣式 */
    </style>
</head>
<body>
    <!-- AsciiEffect.domElement 將被動態添加 -->
    <script type="importmap">
        <!-- Three.js CDN 配置 -->
    </script>
    <script type="module">
        <!-- 應用程式程式碼 -->
    </script>
</body>
</html>
```

### CSS Requirements

```css
body {
    margin: 0;
    padding: 0;
    overflow: hidden;
    background: #000000;
}
```

---

## Event Contracts

### 1. Window Resize Event

| Property | Value |
|----------|-------|
| Event Type | `resize` |
| Target | `window` |
| Handler | `onWindowResize` |
| Debounce | None (即時響應) |

### 2. Mouse Events (via TrackballControls)

| Action | Mouse Button | Result |
|--------|--------------|--------|
| Rotate | Left button drag | 相機繞場景旋轉 |
| Zoom | Scroll wheel | 相機前進/後退 |
| Pan | Right button drag | 相機平移 |

---

## Performance Contract

| Metric | Target | Measurement |
|--------|--------|-------------|
| Frame Rate | ≥ 30 fps | requestAnimationFrame 頻率 |
| Initial Load | < 3 秒 | DOMContentLoaded → 首次渲染 |
| Resize Response | < 500ms | resize 事件 → 渲染更新 |
