# Component Interface: WebGPU Custom Lighting Model

**Feature**: 13-webgpu-lights-custom  
**Date**: 2025-12-01  
**Source**: [data-model.md](../data-model.md)

## Module Structure

```
index.html
├── <style>          # CSS 樣式
├── <div id="info">  # 標題資訊
├── <div id="loading"> # 載入動畫
├── <div id="fallback"> # WebGPU 不支援時的 fallback
└── <script type="module">
    ├── imports
    ├── CustomLightingModel class
    ├── Global variables
    ├── init() function
    ├── onWindowResize() function
    └── animate() function
```

## Import Dependencies

```javascript
// Three.js WebGPU build
import * as THREE from 'three/webgpu';

// TSL functions
import { color, lights } from 'three/tsl';

// Controls
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
```

## Class: CustomLightingModel

```typescript
class CustomLightingModel extends THREE.LightingModel {
    /**
     * 計算直接光照
     * @param params.lightColor - 光源顏色節點
     * @param params.reflectedLight - 反射光累加器
     */
    direct({ lightColor, reflectedLight }): void;
}
```

### Method: direct()

| Parameter | Type | Description |
|-----------|------|-------------|
| lightColor | TSL Node | 當前光源的顏色值 |
| reflectedLight | Object | 包含 directDiffuse 等累加器 |

**Implementation**:
```javascript
direct( { lightColor, reflectedLight } ) {
    reflectedLight.directDiffuse.addAssign( lightColor );
}
```

## Global Variables

| Variable | Type | Scope | Description |
|----------|------|-------|-------------|
| camera | THREE.PerspectiveCamera | module | 主相機 |
| scene | THREE.Scene | module | 主場景 |
| renderer | THREE.WebGPURenderer | module | WebGPU 渲染器 |
| light1 | THREE.PointLight | module | 橙色光源 |
| light2 | THREE.PointLight | module | 藍色光源 |
| light3 | THREE.PointLight | module | 綠色光源 |

## Function: init()

初始化整個場景。

```typescript
function init(): void
```

### Responsibilities

1. 建立 PerspectiveCamera
2. 建立 Scene
3. 建立三個 PointLight（使用 addLight 工廠函式）
4. 建立選擇性光源節點
5. 產生 50 萬隨機點
6. 建立 PointsNodeMaterial 並設定光照模型
7. 建立 Points 物件
8. 初始化 WebGPURenderer
9. 設定 OrbitControls
10. 綁定 resize 事件

### Internal: addLight(hexColor)

```typescript
function addLight(hexColor: number): THREE.PointLight
```

| Parameter | Type | Description |
|-----------|------|-------------|
| hexColor | number | 光源顏色（十六進位） |

**Returns**: 配置好的 PointLight（已附加視覺指示 Mesh）

**Implementation**:
```javascript
const addLight = ( hexColor ) => {
    // 自發光材質
    const material = new THREE.NodeMaterial();
    material.colorNode = color( hexColor );
    material.lightsNode = lights(); // 忽略場景光源

    // 小球 mesh
    const mesh = new THREE.Mesh( sphereGeometry, material );

    // 點光源
    const light = new THREE.PointLight( hexColor, 0.1, 1 );
    light.add( mesh );

    scene.add( light );
    return light;
};
```

## Function: onWindowResize()

處理視窗大小變更。

```typescript
function onWindowResize(): void
```

### Responsibilities

1. 更新 camera.aspect
2. 呼叫 camera.updateProjectionMatrix()
3. 呼叫 renderer.setSize()

## Function: animate()

動畫迴圈函式。

```typescript
function animate(): void
```

### Responsibilities

1. 計算當前時間
2. 更新三個光源位置（使用三角函數）
3. 更新場景旋轉
4. 呼叫 renderer.render()

**Animation Formula**:
```javascript
const time = Date.now() * 0.001;
const scale = 0.5;

light1.position.x = Math.sin( time * 0.7 ) * scale;
light1.position.y = Math.cos( time * 0.5 ) * scale;
light1.position.z = Math.cos( time * 0.3 ) * scale;

light2.position.x = Math.cos( time * 0.3 ) * scale;
light2.position.y = Math.sin( time * 0.5 ) * scale;
light2.position.z = Math.sin( time * 0.7 ) * scale;

light3.position.x = Math.sin( time * 0.7 ) * scale;
light3.position.y = Math.cos( time * 0.3 ) * scale;
light3.position.z = Math.sin( time * 0.5 ) * scale;

scene.rotation.y = time * 0.1;
```

## HTML Structure

### Info Panel

```html
<div id="info">
    <h1>WebGPU Custom Lighting Model</h1>
    <p>Custom lighting model with selective lights</p>
</div>
```

### Loading Indicator

```html
<div id="loading">
    <div class="spinner"></div>
    <p>Loading...</p>
</div>
```

### Fallback (WebGPU not supported)

```html
<div id="fallback" style="display: none;">
    <img src="assets/fallback.png" alt="WebGPU Custom Lighting Preview">
    <h2>WebGPU Not Supported</h2>
    <p>This example requires a WebGPU-compatible browser.</p>
    <p>Please use Chrome 113+, Edge 113+, or another WebGPU-enabled browser.</p>
</div>
```

## CSS Classes

| Class | Purpose |
|-------|---------|
| `#info` | 頂部標題區域 |
| `#loading` | 載入中動畫容器 |
| `#loading.hidden` | 隱藏載入動畫 |
| `.spinner` | 旋轉載入動畫 |
| `#fallback` | WebGPU 不支援時的 fallback 區域 |

## Event Handlers

| Event | Target | Handler |
|-------|--------|---------|
| resize | window | onWindowResize |
| (internal) | renderer | setAnimationLoop(animate) |

## WebGPU Detection

```javascript
if ( !navigator.gpu ) {
    document.getElementById( 'loading' ).style.display = 'none';
    document.getElementById( 'fallback' ).style.display = 'flex';
    return; // 終止初始化
}
```

## Import Map Configuration

```html
<script type="importmap">
{
    "imports": {
        "three": "https://unpkg.com/three@0.170.0/build/three.webgpu.js",
        "three/webgpu": "https://unpkg.com/three@0.170.0/build/three.webgpu.js",
        "three/tsl": "https://unpkg.com/three@0.170.0/build/three.tsl.js",
        "three/addons/": "https://unpkg.com/three@0.170.0/examples/jsm/"
    }
}
</script>
```
