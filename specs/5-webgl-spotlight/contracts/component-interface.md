````markdown
# Component Interface: WebGL Spotlight 互動展示

**Feature**: `5-webgl-spotlight`  
**Date**: 2025-11-28  
**Status**: Complete

## 元件概覽

本範例採用單一 HTML 檔案結構，所有邏輯內嵌於 `<script type="module">` 標籤中。以下定義各功能模組的介面規格。

---

## 1. 初始化模組

### init()

初始化整個場景，包括渲染器、攝影機、控制器、光源、物件和 GUI。

```typescript
function init(): void
```

**執行順序**:
1. 建立 WebGLRenderer
2. 配置渲染器參數 (toneMapping, shadowMap)
3. 建立 Scene 和 Camera
4. 建立 OrbitControls
5. 載入紋理資源
6. 建立光源 (HemisphereLight, SpotLight)
7. 建立輔助物件 (SpotLightHelper, CameraHelper)
8. 建立地面平面
9. 載入 3D 模型
10. 註冊視窗事件
11. 建立 GUI 控制面板

**觸發時機**: 頁面載入完成 (DOMContentLoaded) 或 WebGL 偵測通過後

---

## 2. 資源載入模組

### loadTextures(basePath: string): Record<string, THREE.Texture | null>

載入所有紋理資源。

```typescript
interface TextureMap {
  none: null;
  'disturb.jpg': THREE.Texture;
  'colors.png': THREE.Texture;
  'uv_grid_opengl.jpg': THREE.Texture;
}

function loadTextures(basePath: string): TextureMap
```

**參數**:
- `basePath`: 紋理基礎路徑 (e.g., `'https://threejs.org/examples/textures/'`)

**回傳**: 紋理物件對映表

**錯誤處理**: 個別紋理載入失敗不影響其他紋理，失敗的紋理對應 `null`

---

### loadLucyModel(scene: THREE.Scene): Promise<THREE.Mesh>

載入 Lucy 3D 模型。

```typescript
function loadLucyModel(scene: THREE.Scene): Promise<THREE.Mesh>
```

**參數**:
- `scene`: 要加入模型的場景

**回傳**: Promise 解析為載入的 Mesh 物件

**錯誤處理**: 載入失敗時自動建立並加入後備球體

---

## 3. 場景建構模組

### createRenderer(container: HTMLElement): THREE.WebGLRenderer

建立並配置 WebGL 渲染器。

```typescript
interface RendererConfig {
  antialias: boolean;        // true
  toneMapping: number;       // THREE.NeutralToneMapping
  toneMappingExposure: number; // 1
  shadowMapEnabled: boolean; // true
  shadowMapType: number;     // THREE.PCFSoftShadowMap
}

function createRenderer(container: HTMLElement): THREE.WebGLRenderer
```

---

### createCamera(): THREE.PerspectiveCamera

建立透視攝影機。

```typescript
interface CameraConfig {
  fov: 40;
  aspect: number;  // window.innerWidth / window.innerHeight
  near: 0.1;
  far: 100;
  position: [7, 4, 1];
}

function createCamera(): THREE.PerspectiveCamera
```

---

### createControls(camera: THREE.Camera, domElement: HTMLElement): OrbitControls

建立軌道控制器。

```typescript
interface ControlsConfig {
  minDistance: 2;
  maxDistance: 10;
  maxPolarAngle: number;  // Math.PI / 2
  target: [0, 1, 0];
}

function createControls(
  camera: THREE.Camera, 
  domElement: HTMLElement
): OrbitControls
```

---

### createSpotLight(textures: TextureMap): THREE.SpotLight

建立並配置聚光燈。

```typescript
interface SpotLightConfig {
  color: 0xffffff;
  intensity: 100;
  position: [2.5, 5, 2.5];
  angle: number;     // Math.PI / 6
  penumbra: 1;
  decay: 2;
  distance: 0;
  castShadow: true;
  shadow: {
    mapSize: [1024, 1024];
    cameraNear: 2;
    cameraFar: 10;
    focus: 1;
    bias: -0.003;
    intensity: 1;
  };
}

function createSpotLight(textures: TextureMap): THREE.SpotLight
```

**回傳**: 配置完成的 SpotLight，附加 `lightHelper` 和 `shadowCameraHelper` 屬性

---

### createGroundPlane(): THREE.Mesh

建立地面平面。

```typescript
interface GroundConfig {
  size: [10, 10];
  color: 0xbcbcbc;
  position: [0, -1, 0];
  rotationX: number;  // -Math.PI / 2
  receiveShadow: true;
}

function createGroundPlane(): THREE.Mesh
```

---

### createFallbackModel(): THREE.Mesh

建立後備幾何體（當 Lucy 模型載入失敗時）。

```typescript
interface FallbackConfig {
  radius: 0.5;
  segments: 32;
  color: 0xcccccc;
  positionY: 0.5;
  castShadow: true;
  receiveShadow: true;
}

function createFallbackModel(): THREE.Mesh
```

---

## 4. GUI 模組

### createGUI(spotLight: THREE.SpotLight, textures: TextureMap): GUI

建立 lil-gui 控制面板。

```typescript
interface GUIParams {
  map: THREE.Texture | null;
  color: number;
  intensity: number;
  distance: number;
  angle: number;
  penumbra: number;
  decay: number;
  focus: number;
  shadowIntensity: number;
  helpers: boolean;
}

interface GUIControlRanges {
  intensity: [0, 500];
  distance: [0, 20];
  angle: [0, number];     // Math.PI / 3
  penumbra: [0, 1];
  decay: [1, 2];
  focus: [0, 1];
  shadowIntensity: [0, 1];
}

function createGUI(
  spotLight: THREE.SpotLight, 
  textures: TextureMap
): GUI
```

**事件綁定**:
- 所有參數變更透過 `.onChange()` 即時更新 SpotLight 屬性
- `helpers` 變更切換 lightHelper 和 shadowCameraHelper 的 visible 屬性

---

## 5. 動畫模組

### animate()

動畫主迴圈。

```typescript
function animate(): void
```

**執行內容**:
1. 計算時間: `time = performance.now() / 3000`
2. 更新聚光燈位置:
   - `spotLight.position.x = Math.cos(time) * 2.5`
   - `spotLight.position.z = Math.sin(time) * 2.5`
3. 更新輔助線: `spotLight.lightHelper.update()`
4. 渲染場景: `renderer.render(scene, camera)`

**註冊方式**: `renderer.setAnimationLoop(animate)`

---

## 6. 事件處理模組

### onWindowResize()

處理視窗大小變更。

```typescript
function onWindowResize(): void
```

**執行內容**:
1. 更新攝影機長寬比: `camera.aspect = window.innerWidth / window.innerHeight`
2. 更新投影矩陣: `camera.updateProjectionMatrix()`
3. 更新渲染器尺寸: `renderer.setSize(window.innerWidth, window.innerHeight)`

**註冊方式**: `window.addEventListener('resize', onWindowResize)`

---

## 7. WebGL 偵測模組

### checkWebGLSupport(): boolean

檢查瀏覽器 WebGL 支援。

```typescript
function checkWebGLSupport(): boolean
```

**實作**: 使用 `WebGL.isWebGL2Available()` 或自訂 canvas 偵測

**降級處理**:
- 回傳 `false` 時顯示靜態圖片 `fallback.png`
- 顯示提示訊息引導使用者更新瀏覽器

---

## HTML 結構介面

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, user-scalable=no, minimum-scale=1.0, maximum-scale=1.0">
  <title>three.js webgl - lights - spotlight</title>
  <style>
    /* 全螢幕樣式 */
    body { margin: 0; overflow: hidden; }
    #info { position: absolute; top: 10px; width: 100%; text-align: center; color: #fff; }
    #info a { color: #4a9; }
    .fallback { /* 降級展示樣式 */ }
  </style>
</head>
<body>
  <div id="info">
    <a href="https://threejs.org" target="_blank">three.js</a> webgl - spotlight
  </div>

  <script type="importmap">
    {
      "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
      }
    }
  </script>

  <script type="module">
    // 主程式碼
  </script>
</body>
</html>
```

---

## 相依性圖

```
                        index.html
                            │
                            ▼
                    ┌───────────────┐
                    │  Import Map   │
                    └───────────────┘
                            │
         ┌──────────────────┼──────────────────┐
         │                  │                  │
         ▼                  ▼                  ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   three.js      │ │   lil-gui       │ │   PLYLoader     │
│   (核心)        │ │   (GUI)         │ │   (模型載入)    │
└─────────────────┘ └─────────────────┘ └─────────────────┘
                            │
                            │
                            ▼
                    ┌───────────────┐
                    │ OrbitControls │
                    │ (攝影機控制)  │
                    └───────────────┘
```

---

## 錯誤處理策略

| 錯誤情境 | 處理方式 | 使用者影響 |
|----------|----------|-----------|
| WebGL 不支援 | 顯示靜態圖片 | 無法互動，可看到展示效果 |
| Lucy 模型載入失敗 | 使用後備球體 | 功能完整，模型外觀不同 |
| 紋理載入失敗 | 使用 null (純色光) | 功能完整，無紋理投射效果 |
| 視窗大小為 0 | 忽略 resize 事件 | 無影響 |

````
