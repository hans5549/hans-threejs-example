# Data Model: WebGL Video Kinect 深度影片點雲視覺化

**Feature**: 10-video-kinect  
**Date**: 2025-12-01  
**Status**: Complete

## 概述

本文件定義 WebGL Video Kinect 範例中的資料結構和狀態管理，包括 Shader uniforms、幾何資料、和運行時狀態。

---

## 核心資料結構

### 1. Shader Uniforms

Shader 使用的統一變數，可透過 JavaScript 和 GUI 即時更新。

```typescript
interface ShaderUniforms {
  map: {
    value: THREE.VideoTexture;  // 深度影片紋理
  };
  width: {
    value: number;  // 640 - 影片寬度（像素）
  };
  height: {
    value: number;  // 480 - 影片高度（像素）
  };
  nearClipping: {
    value: number;  // 850 - 近裁切距離（mm），範圍 1-10000
  };
  farClipping: {
    value: number;  // 4000 - 遠裁切距離（mm），範圍 1-10000
  };
  pointSize: {
    value: number;  // 2 - 點大小（像素），範圍 1-10
  };
  zOffset: {
    value: number;  // 1000 - Z軸偏移（mm），範圍 0-4000
  };
}
```

### 2. 點雲幾何資料

BufferGeometry 中的頂點位置陣列。

```typescript
interface PointCloudGeometry {
  // 頂點位置陣列：每 3 個值為一組 (x, y, z)
  // 總共 640 * 480 * 3 = 921,600 個浮點數
  positions: Float32Array;
  
  // 頂點數量
  count: number;  // 307,200
}

// 初始化邏輯
function initializePositions(width: number, height: number): Float32Array {
  const positions = new Float32Array(width * height * 3);
  for (let i = 0, j = 0; i < positions.length; i += 3, j++) {
    positions[i] = j % width;           // 像素 X 座標 (0-639)
    positions[i + 1] = Math.floor(j / width);  // 像素 Y 座標 (0-479)
    positions[i + 2] = 0;               // Z 預設為 0（在 shader 中計算）
  }
  return positions;
}
```

### 3. 相機狀態

```typescript
interface CameraState {
  // 目標位置（由滑鼠控制）
  targetX: number;
  targetY: number;
  
  // 當前位置（透過插值平滑過渡）
  currentX: number;
  currentY: number;
  currentZ: number;  // 固定為 500
  
  // 注視點（場景中心）
  lookAtTarget: THREE.Vector3;  // (0, 0, -1000)
  
  // 插值因子
  lerpFactor: number;  // 0.05
}
```

### 4. 滑鼠輸入狀態

```typescript
interface MouseState {
  // 原始滑鼠座標
  clientX: number;
  clientY: number;
  
  // 轉換後的目標值（乘以 8 的縮放因子）
  x: number;  // (clientX - windowWidth / 2) * 8
  y: number;  // (clientY - windowHeight / 2) * 8
}
```

---

## Shader 資料流

### Vertex Shader 輸入/輸出

```glsl
// === 輸入 ===
// Attributes (來自 BufferGeometry)
attribute vec3 position;  // 像素座標 (x: 0-639, y: 0-479, z: 0)

// Uniforms
uniform sampler2D map;           // 深度影片紋理
uniform float width;             // 640
uniform float height;            // 480
uniform float nearClipping;      // 近裁切距離
uniform float farClipping;       // 遠裁切距離
uniform float pointSize;         // 點大小
uniform float zOffset;           // Z 軸偏移

// 內建常數
const float XtoZ = 1.11146;      // Kinect 水平視野常數
const float YtoZ = 0.83359;      // Kinect 垂直視野常數

// === 輸出 ===
varying vec2 vUv;                // 傳遞給 Fragment Shader 的 UV 座標
gl_PointSize;                    // 點的渲染大小
gl_Position;                     // 頂點的裁切空間座標
```

### Fragment Shader 輸入/輸出

```glsl
// === 輸入 ===
varying vec2 vUv;                // 來自 Vertex Shader 的 UV 座標
uniform sampler2D map;           // 深度影片紋理

// === 輸出 ===
gl_FragColor;                    // 最終像素顏色 (RGBA)
```

---

## 狀態管理

### 初始化流程

```
1. 載入 HTML Video 元素
   └── 設定 loop, muted, crossOrigin, playsinline
   
2. 建立 VideoTexture
   └── 設定 minFilter: NearestFilter
   └── 設定 generateMipmaps: false
   
3. 建立 BufferGeometry
   └── 計算 640x480 個頂點位置
   └── 設定 position attribute
   
4. 建立 ShaderMaterial
   └── 設定 uniforms
   └── 載入 vertex/fragment shader
   └── 設定 blending: AdditiveBlending
   └── 設定 depthTest: false, depthWrite: false
   
5. 建立 Points
   └── 結合 geometry 和 material
   └── 加入 scene
   
6. 建立 GUI
   └── 綁定 uniforms 參數
   
7. 播放影片
   └── video.play()
```

### 渲染迴圈狀態更新

```
每一幀:
1. 更新相機位置 (lerp)
   camera.position.x += (mouse.x - camera.position.x) * 0.05
   camera.position.y += (-mouse.y - camera.position.y) * 0.05
   
2. 更新相機朝向
   camera.lookAt(center)
   
3. 渲染場景
   renderer.render(scene, camera)
```

---

## 常數與預設值

| 常數 | 值 | 說明 |
|------|------|------|
| VIDEO_WIDTH | 640 | 影片寬度（像素）|
| VIDEO_HEIGHT | 480 | 影片高度（像素）|
| POINT_COUNT | 307,200 | 總點數 (640 * 480) |
| DEFAULT_NEAR_CLIPPING | 850 | 預設近裁切（mm）|
| DEFAULT_FAR_CLIPPING | 4000 | 預設遠裁切（mm）|
| DEFAULT_POINT_SIZE | 2 | 預設點大小（像素）|
| DEFAULT_Z_OFFSET | 1000 | 預設 Z 偏移（mm）|
| CAMERA_Z | 500 | 相機 Z 座標 |
| CENTER_Z | -1000 | 場景中心 Z 座標 |
| MOUSE_SCALE | 8 | 滑鼠移動縮放因子 |
| LERP_FACTOR | 0.05 | 相機插值因子 |
| X_TO_Z | 1.11146 | Kinect 水平 FOV 常數 |
| Y_TO_Z | 0.83359 | Kinect 垂直 FOV 常數 |
| POINT_ALPHA | 0.2 | 點的透明度 |

---

## 型別關係圖

```
┌─────────────────────────────────────────────────────────────────┐
│                        Application State                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │ MouseState   │───▶│ CameraState  │───▶│ Camera       │      │
│  │ clientX/Y    │    │ target/lerp  │    │ position/    │      │
│  │ x/y          │    │              │    │ lookAt       │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
│                                                 │                │
│                                                 ▼                │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │ VideoElement │───▶│ VideoTexture │───▶│ Uniforms     │      │
│  │ src: kinect  │    │ NearestFilter│    │ map, width,  │      │
│  │ loop, muted  │    │ no mipmaps   │    │ height, etc. │      │
│  └──────────────┘    └──────────────┘    └──────┬───────┘      │
│                                                 │                │
│                                                 ▼                │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │ PointCloud   │───▶│ Shader       │───▶│ Material     │      │
│  │ Geometry     │    │ Material     │    │ Additive     │      │
│  │ 307,200 pts  │    │ VS + FS      │    │ Blending     │      │
│  └──────────────┘    └──────────────┘    └──────┬───────┘      │
│                                                 │                │
│                                                 ▼                │
│  ┌──────────────┐                        ┌──────────────┐      │
│  │ GUI          │───────────────────────▶│ Points       │      │
│  │ lil-gui      │                        │ Mesh Object  │      │
│  │ param ctrl   │                        │              │      │
│  └──────────────┘                        └──────────────┘      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
