# Component Interface: WebGPU Volume Caustics

**Feature**: 4-webgpu-volume-caustics  
**Date**: 2025-11-28

## Overview

本文件定義前端元件的介面契約。由於這是單頁 HTML 應用，主要定義 JavaScript 模組的函式簽章與事件介面。

## Module Exports

### Main Application Module

```typescript
// 應用程式主要介面（概念性 TypeScript 定義）

interface AppConfig {
  containerId?: string;        // 渲染容器 ID，預設 document.body
  enableInspector?: boolean;   // 是否啟用 Inspector，預設 false
}

interface App {
  init(): Promise<void>;       // 初始化應用程式
  dispose(): void;             // 清理資源
}
```

### WebGPU Capability Check

```typescript
interface WebGPUSupport {
  isSupported: boolean;
  adapter?: GPUAdapter;
  device?: GPUDevice;
  errorMessage?: string;
}

function checkWebGPUSupport(): Promise<WebGPUSupport>;
```

**Response Examples**:

支援 WebGPU:
```javascript
{
  isSupported: true,
  adapter: GPUAdapter,
  device: GPUDevice
}
```

不支援 WebGPU:
```javascript
{
  isSupported: false,
  errorMessage: "WebGPU is not supported in this browser"
}
```

## TSL Shader Interfaces

### Caustic Effect Shader

```typescript
interface CausticShaderParams {
  causticMap: THREE.Texture;      // 焦散紋理
  ior: number;                    // 折射率
  causticOcclusion: number;       // 遮蔽係數
  causticColor: THREE.Color;      // 焦散顏色
}

// TSL 函式簽章
type CausticEffect = () => TSL.Node;
```

**輸入參數**:
- `positionViewDirection`: 視角方向向量
- `normalView`: 表面法線（視角空間）
- `causticMap`: 焦散紋理

**輸出**:
- `vec3`: RGB 顏色值，用於 castShadowNode

### Emissive Shader

```typescript
interface EmissiveShaderParams {
  spotLight: THREE.SpotLight;     // 聚光燈參考
  causticEffect: TSL.Node;        // 焦散效果節點
}

type EmissiveEffect = () => TSL.Node;
```

**輸入參數**:
- `lightViewPosition(spotLight)`: 光源視角位置
- `positionView`: 當前片段視角位置
- `positionViewDirection`: 視角方向

**輸出**:
- `vec3`: 發光顏色值

### Volumetric Scattering Shader

```typescript
interface ScatteringParams {
  noiseTexture3D: THREE.Data3DTexture;  // 3D 噪聲紋理
  smokeAmount: number;                   // 煙霧濃度
}

type ScatteringFunction = (params: { positionRay: TSL.Node }) => TSL.Node;
```

**輸入參數**:
- `positionRay`: 當前射線位置
- `time`: 時間節點（動畫用）

**輸出**:
- `float`: 散射密度值

## Event Interfaces

### Window Events

| Event | Handler | Description |
|-------|---------|-------------|
| `resize` | `onWindowResize()` | 視窗大小變更時更新相機和渲染器 |
| `DOMContentLoaded` | `init()` | DOM 載入完成後初始化 |

### OrbitControls Events

| Event | Behavior |
|-------|----------|
| `mousedown` + `mousemove` | 旋轉相機視角 |
| `wheel` | 縮放相機距離 |
| `touchstart` + `touchmove` | 觸控旋轉 |
| `pinch` | 觸控縮放 |

**Controls Configuration**:
```typescript
interface OrbitControlsConfig {
  target: THREE.Vector3;      // 焦點位置
  maxDistance: number;        // 最大縮放距離
  enableDamping?: boolean;    // 啟用阻尼
  dampingFactor?: number;     // 阻尼係數
}
```

## Render Pipeline

### Pass Order

```
1. Scene Pass (scenePass)
   ├── Input: scene, camera
   ├── Output: color texture, depth texture
   └── Layer: all layers

2. Volumetric Pass (volumetricPass)
   ├── Input: scene, camera
   ├── Output: volumetric color texture
   ├── Layer: LAYER_VOLUMETRIC_LIGHTING only
   └── Resolution: 0.5x

3. Bloom Pass (bloomPass)
   ├── Input: volumetricPass output
   └── Output: blurred volumetric texture

4. Composite
   ├── Formula: scenePass + bloomPass * intensity
   └── Output: final frame
```

### Layer Configuration

| Layer ID | Name | Contents |
|----------|------|----------|
| 0 | Default | Duck, Ground, SpotLight |
| 10 | Volumetric | VolumetricMesh only |

## Error Handling

### Error Types

```typescript
enum AppErrorType {
  WEBGPU_NOT_SUPPORTED = 'WEBGPU_NOT_SUPPORTED',
  MODEL_LOAD_FAILED = 'MODEL_LOAD_FAILED',
  TEXTURE_LOAD_FAILED = 'TEXTURE_LOAD_FAILED',
  SHADER_COMPILE_ERROR = 'SHADER_COMPILE_ERROR'
}

interface AppError {
  type: AppErrorType;
  message: string;
  details?: unknown;
}
```

### Fallback UI Contract

當 WebGPU 不支援時，顯示以下 UI：

```html
<div id="fallback">
  <img src="fallback-preview.jpg" alt="WebGPU Volume Caustics Preview">
  <p>Your browser does not support WebGPU.</p>
  <p>Please use Chrome 113+ or Edge 113+ for the full experience.</p>
</div>
```

## Animation Loop Contract

```typescript
interface AnimationState {
  isRunning: boolean;
  lastFrameTime: number;
  frameCount: number;
}

function animate(): void;
// 每幀執行:
// 1. 旋轉鴨子模型 (mesh.rotation.y -= 0.01)
// 2. 更新 OrbitControls
// 3. 呼叫 postProcessing.render()
```

## Import Map Contract

應用程式依賴以下模組映射：

```json
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.webgpu.js",
    "three/webgpu": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.webgpu.js",
    "three/tsl": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.tsl.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
  }
}
```
