# Component Interface: WebGL Custom Attributes Lines

**Feature**: 12-custom-attributes-lines  
**Date**: 2025-12-01  
**Status**: Complete

## 模組結構

本功能採用單一 HTML 檔案結構，所有程式碼內嵌於 `index.html`。以下定義各邏輯模組的介面。

---

## 1. 全域變數介面

### 核心渲染物件

```typescript
// Three.js 核心物件
let renderer: THREE.WebGLRenderer;
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;

// 功能物件
let line: THREE.Line;
let uniforms: {
  amplitude: { value: number };
  opacity: { value: number };
  color: { value: THREE.Color };
};

// 輔助物件
let stats: Stats;
```

---

## 2. 初始化函式介面

### FontLoader 載入

```typescript
/**
 * 載入字型並初始化場景
 * @description 使用 FontLoader 載入 Helvetiker Bold 字型
 * @callback onLoad 字型載入成功時呼叫 init(font)
 * @callback onError 字型載入失敗時呼叫 initWithFallback()
 */
function loadFont(): void;
```

### 主要初始化（使用 TextGeometry）

```typescript
/**
 * 使用字型初始化場景
 * @param font - 載入的 Font 物件
 * @description 建立 TextGeometry、ShaderMaterial、Line 並加入場景
 */
function init(font: THREE.Font): void;
```

### 備用初始化（使用 BoxGeometry）

```typescript
/**
 * 使用備用幾何體初始化場景
 * @description 當字型載入失敗時呼叫，使用 BoxGeometry 替代
 */
function initWithFallback(): void;
```

---

## 3. 幾何體設定介面

### 建立 TextGeometry

```typescript
/**
 * 建立 TextGeometry
 * @param font - 載入的字型物件
 * @returns 置中的 TextGeometry
 */
function createTextGeometry(font: THREE.Font): THREE.TextGeometry;

// 參數配置
const TEXT_CONFIG = {
  text: 'three.js',
  size: 50,
  depth: 15,
  curveSegments: 10,
  bevelThickness: 5,
  bevelSize: 1.5,
  bevelEnabled: true,
  bevelSegments: 10
};
```

### 建立備用 BoxGeometry

```typescript
/**
 * 建立備用 BoxGeometry
 * @returns 用於 fallback 的 BoxGeometry
 */
function createFallbackGeometry(): THREE.BoxGeometry;

// 參數配置
const FALLBACK_CONFIG = {
  width: 100,
  height: 100,
  depth: 100,
  widthSegments: 10,
  heightSegments: 10,
  depthSegments: 10
};
```

### 設定幾何體屬性

```typescript
/**
 * 為幾何體設定自訂 buffer attributes
 * @param geometry - TextGeometry 或 BoxGeometry
 * @description 建立並附加 displacement 和 customColor attributes
 */
function setupGeometryAttributes(geometry: THREE.BufferGeometry): void;
```

---

## 4. 材質介面

### 建立 ShaderMaterial

```typescript
/**
 * 建立 ShaderMaterial
 * @returns 配置好的 ShaderMaterial
 */
function createShaderMaterial(): THREE.ShaderMaterial;

// Uniforms 初始值
const INITIAL_UNIFORMS = {
  amplitude: { value: 5.0 },
  opacity: { value: 0.3 },
  color: { value: new THREE.Color(0xffffff) }
};

// Material 配置
const MATERIAL_CONFIG = {
  blending: THREE.AdditiveBlending,
  depthTest: false,
  transparent: true
};
```

---

## 5. 動畫介面

### 動畫主迴圈

```typescript
/**
 * 動畫主函式
 * @description 由 setAnimationLoop 呼叫，處理所有動畫更新和渲染
 */
function animate(): void;
```

### 渲染函式

```typescript
/**
 * 渲染函式
 * @description 更新 uniforms、attributes 和物件旋轉，然後渲染場景
 */
function render(): void;

// 動畫參數
const ANIMATION_CONFIG = {
  rotationSpeed: 0.25,           // line.rotation.y = rotationSpeed * time
  amplitudeFrequency: 0.5,       // sin(amplitudeFrequency * time)
  hslOffsetSpeed: 0.0005,        // color.offsetHSL(hslOffsetSpeed, 0, 0)
  displacementDelta: 0.3         // displacement change per frame
};
```

---

## 6. 事件處理介面

### 視窗調整

```typescript
/**
 * 處理視窗大小調整
 * @description 更新相機 aspect ratio 和渲染器尺寸
 */
function onWindowResize(): void;
```

---

## 7. Shader 程式碼介面

### Vertex Shader

```glsl
// ID: vertexshader
// 位置: <script type="x-shader/x-vertex" id="vertexshader">

uniform float amplitude;

attribute vec3 displacement;
attribute vec3 customColor;

varying vec3 vColor;

void main() {
    vec3 newPosition = position + amplitude * displacement;
    vColor = customColor;
    gl_Position = projectionMatrix * modelViewMatrix * vec4(newPosition, 1.0);
}
```

### Fragment Shader

```glsl
// ID: fragmentshader
// 位置: <script type="x-shader/x-fragment" id="fragmentshader">

uniform vec3 color;
uniform float opacity;

varying vec3 vColor;

void main() {
    gl_FragColor = vec4(vColor * color, opacity);
}
```

---

## 8. HTML 結構介面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>three.js webgl - custom attributes [lines]</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, ...">
    <style>/* CSS 樣式 */</style>
</head>
<body>
    <div id="container"></div>
    <div id="info">
        <a href="https://threejs.org">three.js</a> - custom attributes example
    </div>
    
    <!-- Shaders -->
    <script type="x-shader/x-vertex" id="vertexshader">...</script>
    <script type="x-shader/x-fragment" id="fragmentshader">...</script>
    
    <!-- Import Map -->
    <script type="importmap">{ "imports": { ... } }</script>
    
    <!-- Main Module -->
    <script type="module">/* JavaScript 程式碼 */</script>
</body>
</html>
```

---

## 9. 依賴套件介面

### Import Map 配置

```json
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.174.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.174.0/examples/jsm/"
  }
}
```

### 必要匯入

```typescript
import * as THREE from 'three';
import { FontLoader } from 'three/addons/loaders/FontLoader.js';
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';
import Stats from 'three/addons/libs/stats.module.js';
```

---

## 10. 錯誤處理介面

### WebGL 支援檢查

```typescript
/**
 * 檢查 WebGL 支援
 * @returns 是否支援 WebGL
 * @description 若不支援則顯示 fallback 圖片
 */
function checkWebGLSupport(): boolean;
```

### 字型載入錯誤

```typescript
/**
 * 字型載入錯誤處理
 * @param error - 錯誤物件
 * @description 記錄警告並使用 fallback geometry
 */
function handleFontLoadError(error: Error): void;
```
