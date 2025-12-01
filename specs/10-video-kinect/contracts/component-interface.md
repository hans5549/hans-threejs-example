# Component Interface: WebGL Video Kinect

**Feature**: 10-video-kinect  
**Date**: 2025-12-01  
**Status**: Complete

## 概述

本文件定義 WebGL Video Kinect 範例中各元件的介面契約，包括初始化、事件處理、和公開 API。

---

## 1. 主程式初始化介面

### init()

初始化整個應用程式。

```typescript
/**
 * 應用程式主要初始化函式
 * 建立場景、相機、渲染器、點雲、GUI
 */
function init(): void;
```

**前置條件:**
- DOM 已完全載入
- WebGL 支援可用

**後置條件:**
- Scene、Camera、Renderer 已建立
- 點雲材質和幾何已建立
- GUI 控制面板已顯示
- 影片開始播放

---

## 2. 影片元素介面

### HTML Video Element Configuration

```html
<video id="video" loop muted crossOrigin="anonymous" playsinline>
  <source src="textures/kinect.webm" type="video/webm">
  <source src="textures/kinect.mp4" type="video/mp4">
</video>
```

### Video API Usage

```typescript
interface VideoElement {
  // 取得影片元素
  getElementById(id: 'video'): HTMLVideoElement;
}

// 使用範例
const video: HTMLVideoElement = document.getElementById('video');
video.play();  // 開始播放
```

---

## 3. VideoTexture 介面

```typescript
interface VideoTextureConfig {
  source: HTMLVideoElement;
  options: {
    minFilter: THREE.NearestFilter;
    generateMipmaps: false;
  };
}

// 建立
const texture = new THREE.VideoTexture(video);
texture.minFilter = THREE.NearestFilter;
texture.generateMipmaps = false;
```

---

## 4. ShaderMaterial 介面

### Uniforms 介面

```typescript
interface DepthShaderUniforms {
  map: { value: THREE.Texture };
  width: { value: number };
  height: { value: number };
  nearClipping: { value: number };
  farClipping: { value: number };
  pointSize: { value: number };
  zOffset: { value: number };
}
```

### Material 建立

```typescript
interface ShaderMaterialConfig {
  uniforms: DepthShaderUniforms;
  vertexShader: string;
  fragmentShader: string;
  blending: THREE.AdditiveBlending;
  depthTest: false;
  depthWrite: false;
  transparent: true;
}

const material = new THREE.ShaderMaterial({
  uniforms,
  vertexShader,
  fragmentShader,
  blending: THREE.AdditiveBlending,
  depthTest: false,
  depthWrite: false,
  transparent: true
});
```

---

## 5. Vertex Shader 介面

### 輸入

```glsl
// Attributes (自動由 Three.js 提供)
attribute vec3 position;

// Uniforms
uniform sampler2D map;
uniform float width;
uniform float height;
uniform float nearClipping;
uniform float farClipping;
uniform float pointSize;
uniform float zOffset;

// Matrices (自動由 Three.js 提供)
uniform mat4 modelViewMatrix;
uniform mat4 projectionMatrix;
```

### 輸出

```glsl
varying vec2 vUv;  // UV 座標傳遞給 Fragment Shader
gl_PointSize;      // 點大小
gl_Position;       // 裁切空間座標
```

### 計算邏輯

```glsl
void main() {
  // 1. 計算 UV 座標
  vUv = vec2(position.x / width, position.y / height);
  
  // 2. 從深度紋理取樣
  vec4 color = texture2D(map, vUv);
  
  // 3. 解碼深度值 (RGB -> 單一深度值)
  float depth = (color.r + color.g + color.b) / 3.0;
  
  // 4. 深度映射與裁切
  float z = (1.0 - depth) * (farClipping - nearClipping) + nearClipping;
  
  // 5. 應用 Kinect FOV 轉換
  vec4 pos = vec4(
    (position.x / width - 0.5) * z * XtoZ,
    (position.y / height - 0.5) * z * YtoZ,
    -z + zOffset,
    1.0
  );
  
  // 6. 設定輸出
  gl_PointSize = pointSize;
  gl_Position = projectionMatrix * modelViewMatrix * pos;
}
```

---

## 6. Fragment Shader 介面

### 輸入

```glsl
varying vec2 vUv;
uniform sampler2D map;
```

### 輸出

```glsl
gl_FragColor;  // vec4(r, g, b, a)
```

### 計算邏輯

```glsl
void main() {
  vec4 color = texture2D(map, vUv);
  gl_FragColor = vec4(color.r, color.g, color.b, 0.2);
}
```

---

## 7. 事件處理介面

### 滑鼠移動事件

```typescript
interface MouseMoveHandler {
  (event: MouseEvent): void;
}

function onDocumentMouseMove(event: MouseEvent): void {
  // 計算滑鼠相對於視窗中心的偏移，並放大 8 倍
  mouse.x = (event.clientX - windowHalfX) * 8;
  mouse.y = (event.clientY - windowHalfY) * 8;
}

// 註冊
document.addEventListener('mousemove', onDocumentMouseMove);
```

### 視窗調整事件

```typescript
interface WindowResizeHandler {
  (): void;
}

function onWindowResize(): void {
  // 更新視窗中心
  windowHalfX = window.innerWidth / 2;
  windowHalfY = window.innerHeight / 2;
  
  // 更新相機長寬比
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  
  // 更新渲染器尺寸
  renderer.setSize(window.innerWidth, window.innerHeight);
}

// 註冊
window.addEventListener('resize', onWindowResize);
```

---

## 8. GUI 介面

### lil-gui 控制項配置

```typescript
interface GUIConfig {
  nearClipping: {
    min: 1;
    max: 10000;
    step: 1;
    default: 850;
  };
  farClipping: {
    min: 1;
    max: 10000;
    step: 1;
    default: 4000;
  };
  pointSize: {
    min: 1;
    max: 10;
    step: 1;
    default: 2;
  };
  zOffset: {
    min: 0;
    max: 4000;
    step: 1;
    default: 1000;
  };
}
```

### GUI 建立程式碼

```typescript
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';

const gui = new GUI();

// 為每個 uniform 建立滑桿
gui.add(uniforms.nearClipping, 'value', 1, 10000, 1).name('Near Clipping');
gui.add(uniforms.farClipping, 'value', 1, 10000, 1).name('Far Clipping');
gui.add(uniforms.pointSize, 'value', 1, 10, 1).name('Point Size');
gui.add(uniforms.zOffset, 'value', 0, 4000, 1).name('Z Offset');
```

---

## 9. 動畫迴圈介面

### animate()

```typescript
/**
 * 動畫迴圈主函式
 * 由 requestAnimationFrame 呼叫
 */
function animate(): void {
  requestAnimationFrame(animate);
  render();
}
```

### render()

```typescript
/**
 * 渲染單一幀
 * 更新相機位置並渲染場景
 */
function render(): void {
  // 相機平滑跟隨滑鼠
  camera.position.x += (mouse.x - camera.position.x) * 0.05;
  camera.position.y += (-mouse.y - camera.position.y) * 0.05;
  camera.lookAt(center);
  
  // 渲染
  renderer.render(scene, camera);
}
```

---

## 10. 完整元件依賴圖

```
                    ┌─────────────┐
                    │   init()    │
                    └─────┬───────┘
                          │
         ┌────────────────┼────────────────┐
         │                │                │
         ▼                ▼                ▼
  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
  │   Scene      │ │   Camera     │ │  Renderer    │
  │   Setup      │ │   Setup      │ │   Setup      │
  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
         │                │                │
         │                │                │
         ▼                │                │
  ┌──────────────┐        │                │
  │   Video      │        │                │
  │   Element    │        │                │
  └──────┬───────┘        │                │
         │                │                │
         ▼                │                │
  ┌──────────────┐        │                │
  │  VideoTexture│        │                │
  └──────┬───────┘        │                │
         │                │                │
         ▼                │                │
  ┌──────────────┐        │                │
  │  Uniforms    │◀───────┼────────────────┤
  └──────┬───────┘        │                │
         │                │                │
         ▼                │                │
  ┌──────────────┐        │                │
  │  Shader      │        │                │
  │  Material    │        │                │
  └──────┬───────┘        │                │
         │                │                │
         ▼                │                │
  ┌──────────────┐        │                │
  │  BufferGeo   │        │                │
  │  + Points    │        │                │
  └──────┬───────┘        │                │
         │                │                │
         │        ┌───────┴───────┐        │
         │        │               │        │
         ▼        ▼               ▼        ▼
      ┌─────────────────────────────────────┐
      │            animate()                 │
      │  ┌─────────────────────────────┐    │
      │  │          render()           │    │
      │  │  • Camera position lerp     │    │
      │  │  • renderer.render()        │    │
      │  └─────────────────────────────┘    │
      └─────────────────────────────────────┘
              ▲                ▲
              │                │
      ┌───────┴──────┐ ┌───────┴──────┐
      │   Mouse      │ │   Window     │
      │   Events     │ │   Resize     │
      └──────────────┘ └──────────────┘
              ▲
              │
      ┌───────┴──────┐
      │   GUI        │
      │   Controls   │
      └──────────────┘
```
