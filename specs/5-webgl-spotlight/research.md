````markdown
# Research: WebGL Spotlight 互動展示

**Feature**: `5-webgl-spotlight`  
**Date**: 2025-11-28  
**Status**: Complete

## 研究摘要

本研究文件整理了實作 Three.js WebGL Spotlight 範例所需的技術細節，基於 Three.js 官方範例 `webgl_lights_spotlight` 進行分析。

---

## 1. SpotLight 光源設定

### Decision: 使用 THREE.SpotLight 配置完整參數

**Rationale**: SpotLight 是 Three.js 中功能最豐富的光源類型，支援位置、方向、顏色、強度、角度、衰減、陰影投射和紋理投射。

**Alternatives considered**:
- PointLight - 無法產生聚光效果和紋理投射
- DirectionalLight - 無法產生圓錐形照明區域

### 實作方式

```javascript
const spotLight = new THREE.SpotLight(0xffffff, 100);
spotLight.position.set(2.5, 5, 2.5);
spotLight.angle = Math.PI / 6;           // 聚光燈角度 (30°)
spotLight.penumbra = 1;                   // 邊緣模糊度 (0-1)
spotLight.decay = 2;                      // 衰減係數 (物理正確為 2)
spotLight.distance = 0;                   // 0 = 無距離限制

spotLight.castShadow = true;
spotLight.shadow.mapSize.width = 1024;
spotLight.shadow.mapSize.height = 1024;
spotLight.shadow.camera.near = 2;
spotLight.shadow.camera.far = 10;
spotLight.shadow.focus = 1;
spotLight.shadow.bias = -0.003;
spotLight.shadow.intensity = 1;

scene.add(spotLight);
```

### SpotLight 屬性說明

| 屬性 | 類型 | 範圍 | 說明 |
|------|------|------|------|
| `color` | Color | - | 光源顏色 |
| `intensity` | number | 0-500 | 光源強度 (candela) |
| `distance` | number | 0-20 | 光照距離限制，0 為無限 |
| `angle` | number | 0-π/3 | 聚光燈錐角 (弧度) |
| `penumbra` | number | 0-1 | 邊緣模糊度 |
| `decay` | number | 1-2 | 光照衰減率 |
| `map` | Texture | - | 紋理投射貼圖 |

---

## 2. 紋理投射功能

### Decision: 使用 SpotLight.map 屬性配置紋理

**Rationale**: SpotLight 原生支援 map 屬性，可將紋理圖像投射到照亮的表面，類似投影機效果。

### 實作方式

```javascript
const loader = new THREE.TextureLoader().setPath('https://threejs.org/examples/textures/');
const filenames = ['disturb.jpg', 'colors.png', 'uv_grid_opengl.jpg'];

const textures = { none: null };

for (const filename of filenames) {
  const texture = loader.load(filename);
  texture.minFilter = THREE.LinearFilter;
  texture.magFilter = THREE.LinearFilter;
  texture.generateMipmaps = false;
  texture.colorSpace = THREE.SRGBColorSpace;
  textures[filename] = texture;
}

// 套用紋理
spotLight.map = textures['disturb.jpg'];
```

### 紋理配置要點

| 設定 | 值 | 說明 |
|------|-----|------|
| `minFilter` | LinearFilter | 縮小時使用線性過濾 |
| `magFilter` | LinearFilter | 放大時使用線性過濾 |
| `generateMipmaps` | false | 關閉 mipmap 以節省記憶體 |
| `colorSpace` | SRGBColorSpace | 使用 sRGB 色彩空間確保顏色正確 |

### 紋理 CDN 路徑

```
https://threejs.org/examples/textures/disturb.jpg
https://threejs.org/examples/textures/colors.png
https://threejs.org/examples/textures/uv_grid_opengl.jpg
```

---

## 3. PCF Soft Shadow 陰影渲染

### Decision: 使用 PCFSoftShadowMap 產生軟陰影

**Rationale**: PCFSoftShadowMap 在效能和品質之間取得平衡，產生平滑的陰影邊緣。

**Alternatives considered**:
- BasicShadowMap - 品質較低，邊緣鋸齒明顯
- PCFShadowMap - 品質中等，邊緣略硬
- VSMShadowMap - 品質高但效能開銷大

### 實作方式

```javascript
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
```

### 陰影相關配置

```javascript
// 光源陰影設定
spotLight.shadow.mapSize.width = 1024;    // 陰影貼圖解析度
spotLight.shadow.mapSize.height = 1024;
spotLight.shadow.camera.near = 2;          // 陰影攝影機近平面
spotLight.shadow.camera.far = 10;          // 陰影攝影機遠平面
spotLight.shadow.focus = 1;                // 陰影聚焦 (0-1)
spotLight.shadow.bias = -0.003;            // 偏移以避免陰影痤瘡
spotLight.shadow.intensity = 1;            // 陰影強度 (0-1)

// 物件陰影設定
mesh.castShadow = true;     // 投射陰影
mesh.receiveShadow = true;  // 接收陰影
```

---

## 4. PLYLoader 載入 3D 模型

### Decision: 使用 PLYLoader 從 CDN 載入 Lucy 模型

**Rationale**: PLY 格式適合高精度掃描模型，Three.js 官方提供的 Lucy 模型是經典的光照測試模型。

### 實作方式

```javascript
import { PLYLoader } from 'three/addons/loaders/PLYLoader.js';

const plyLoader = new PLYLoader();
plyLoader.load(
  'https://threejs.org/examples/models/ply/binary/Lucy100k.ply',
  function (geometry) {
    // 成功回調
    geometry.scale(0.0024, 0.0024, 0.0024);  // 縮放至適當大小
    geometry.computeVertexNormals();          // 計算法線以正確光照

    const material = new THREE.MeshLambertMaterial();
    const mesh = new THREE.Mesh(geometry, material);
    mesh.rotation.y = -Math.PI / 2;
    mesh.position.y = 0.8;
    mesh.castShadow = true;
    mesh.receiveShadow = true;
    scene.add(mesh);
  },
  undefined,  // 進度回調 (可選)
  function (error) {
    // 錯誤回調 - 使用後備幾何體
    console.warn('Lucy model failed to load, using fallback sphere');
    const fallbackGeometry = new THREE.SphereGeometry(0.5, 32, 32);
    const fallbackMaterial = new THREE.MeshLambertMaterial({ color: 0xcccccc });
    const fallbackMesh = new THREE.Mesh(fallbackGeometry, fallbackMaterial);
    fallbackMesh.position.y = 0.5;
    fallbackMesh.castShadow = true;
    fallbackMesh.receiveShadow = true;
    scene.add(fallbackMesh);
  }
);
```

### 模型 CDN 路徑

```
https://threejs.org/examples/models/ply/binary/Lucy100k.ply
```

---

## 5. lil-gui 控制面板

### Decision: 使用 lil-gui 建立互動式 GUI

**Rationale**: lil-gui 是 dat.gui 的現代替代品，體積小、效能好，是 Three.js 官方範例使用的標準 GUI 函式庫。

### 實作方式

```javascript
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';

const gui = new GUI();

const params = {
  map: textures['disturb.jpg'],
  color: spotLight.color.getHex(),
  intensity: spotLight.intensity,
  distance: spotLight.distance,
  angle: spotLight.angle,
  penumbra: spotLight.penumbra,
  decay: spotLight.decay,
  focus: spotLight.shadow.focus,
  shadowIntensity: spotLight.shadow.intensity,
  helpers: false
};

// 紋理選擇 (下拉選單)
gui.add(params, 'map', textures).onChange((val) => {
  spotLight.map = val;
});

// 顏色選擇器
gui.addColor(params, 'color').onChange((val) => {
  spotLight.color.setHex(val);
});

// 數值滑桿
gui.add(params, 'intensity', 0, 500).onChange((val) => {
  spotLight.intensity = val;
});

gui.add(params, 'distance', 0, 20).onChange((val) => {
  spotLight.distance = val;
});

gui.add(params, 'angle', 0, Math.PI / 3).onChange((val) => {
  spotLight.angle = val;
});

gui.add(params, 'penumbra', 0, 1).onChange((val) => {
  spotLight.penumbra = val;
});

gui.add(params, 'decay', 1, 2).onChange((val) => {
  spotLight.decay = val;
});

gui.add(params, 'focus', 0, 1).onChange((val) => {
  spotLight.shadow.focus = val;
});

gui.add(params, 'shadowIntensity', 0, 1).onChange((val) => {
  spotLight.shadow.intensity = val;
});

// 布林開關
gui.add(params, 'helpers').onChange((val) => {
  spotLight.lightHelper.visible = val;
  spotLight.shadowCameraHelper.visible = val;
});

gui.open();
```

---

## 6. SpotLightHelper 和 CameraHelper

### Decision: 使用內建 Helper 類別

**Rationale**: Three.js 提供現成的輔助視覺化工具，可直接使用無需自行實作。

### 實作方式

```javascript
// 聚光燈輔助線
spotLight.lightHelper = new THREE.SpotLightHelper(spotLight);
spotLight.lightHelper.visible = false;  // 預設隱藏
scene.add(spotLight.lightHelper);

// 陰影攝影機輔助線
spotLight.shadowCameraHelper = new THREE.CameraHelper(spotLight.shadow.camera);
spotLight.shadowCameraHelper.visible = false;  // 預設隱藏
scene.add(spotLight.shadowCameraHelper);

// 在動畫迴圈中更新
function animate() {
  spotLight.lightHelper.update();
  // CameraHelper 不需要手動更新
}
```

---

## 7. Neutral Tone Mapping

### Decision: 使用 NeutralToneMapping 配置

**Rationale**: NeutralToneMapping 是 Three.js r160+ 新增的色調映射，提供自然的 HDR 轉 LDR 效果。

### 實作方式

```javascript
renderer.toneMapping = THREE.NeutralToneMapping;
renderer.toneMappingExposure = 1;
```

---

## 8. WebGL 偵測和降級

### Decision: 使用 WebGL.isWebGL2Available() 偵測

**Rationale**: Three.js 提供內建的 WebGL 偵測方法，可配合降級方案使用。

### 實作方式

```javascript
import WebGL from 'three/addons/capabilities/WebGL.js';

if (WebGL.isWebGL2Available()) {
  // 正常初始化 Three.js 場景
  init();
} else {
  // 顯示降級靜態圖片
  const container = document.getElementById('container');
  container.innerHTML = `
    <div class="fallback">
      <img src="fallback.png" alt="WebGL Spotlight Demo">
      <p>您的瀏覽器不支援 WebGL。請使用現代瀏覽器以獲得最佳體驗。</p>
    </div>
  `;
}
```

---

## 9. 環境光配置

### Decision: 使用 HemisphereLight 作為環境光

**Rationale**: HemisphereLight 模擬天空和地面的環境反射，比 AmbientLight 更自然。

### 實作方式

```javascript
const ambient = new THREE.HemisphereLight(0xffffff, 0x8d8d8d, 0.25);
scene.add(ambient);
```

| 參數 | 值 | 說明 |
|------|-----|------|
| skyColor | 0xffffff | 天空顏色 (白色) |
| groundColor | 0x8d8d8d | 地面顏色 (灰色) |
| intensity | 0.25 | 強度 (較低以突顯聚光燈效果) |

---

## 10. OrbitControls 攝影機控制

### Decision: 使用 OrbitControls 限制參數

**Rationale**: OrbitControls 提供直覺的軌道控制，可配置距離和角度限制。

### 實作方式

```javascript
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const controls = new OrbitControls(camera, renderer.domElement);
controls.minDistance = 2;                    // 最小距離
controls.maxDistance = 10;                   // 最大距離
controls.maxPolarAngle = Math.PI / 2;        // 不低於地平面
controls.target.set(0, 1, 0);                // 觀察目標點
controls.update();
```

---

## 11. 聚光燈動態移動動畫

### Decision: 使用三角函數實現圓形路徑

**Rationale**: 簡單的 cos/sin 函數可產生平滑的圓形軌跡，無需複雜的路徑系統。

### 實作方式

```javascript
function animate() {
  const time = performance.now() / 3000;  // 控制旋轉速度

  spotLight.position.x = Math.cos(time) * 2.5;
  spotLight.position.z = Math.sin(time) * 2.5;
  // Y 軸位置保持固定 (5)

  spotLight.lightHelper.update();  // 更新輔助線

  renderer.render(scene, camera);
}
```

---

## CDN 資源清單

| 資源類型 | URL |
|----------|-----|
| Three.js 核心 | `https://unpkg.com/three@0.160.0/build/three.module.js` |
| Three.js Addons | `https://unpkg.com/three@0.160.0/examples/jsm/` |
| Lucy 模型 | `https://threejs.org/examples/models/ply/binary/Lucy100k.ply` |
| disturb.jpg | `https://threejs.org/examples/textures/disturb.jpg` |
| colors.png | `https://threejs.org/examples/textures/colors.png` |
| uv_grid_opengl.jpg | `https://threejs.org/examples/textures/uv_grid_opengl.jpg` |

````
