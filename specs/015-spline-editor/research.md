# Research: WebGL Geometry Spline Editor

**Feature**: `015-spline-editor`  
**Date**: 2025-12-04  
**Status**: Complete

## 研究摘要

本研究文件整理了實作 Three.js 互動式 Catmull-Rom 曲線編輯器所需的技術細節，基於 Three.js 官方範例 `webgl_geometry_spline_editor` 進行分析。

---

## 1. Three.js CDN 載入方式

### Decision: 使用 Import Map + ES Module

**Rationale**: 這是 Three.js r170+ 推薦的現代載入方式，無需建置工具，直接在瀏覽器中使用 ES Module。

**Alternatives considered**:
- UMD 全域變數方式 - 已過時，不支援 tree-shaking
- npm + bundler - 對於單一展示範例過於複雜

### 實作方式

```html
<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.170.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.170.0/examples/jsm/"
  }
}
</script>

<script type="module">
import * as THREE from 'three';
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { TransformControls } from 'three/addons/controls/TransformControls.js';
</script>
```

---

## 2. CatmullRomCurve3 API

### Decision: 使用 Three.js 內建的 CatmullRomCurve3 類別

**Rationale**: Three.js 提供完整的 Catmull-Rom 曲線實作，支援三種曲線類型和張力參數調整。

### 曲線類型

| curveType | 說明 | 顏色（範例） |
|-----------|------|--------------|
| `catmullrom` | 標準 uniform 參數化，可調整 tension | 紅色 |
| `centripetal` | 避免尖點和自相交，平滑度較佳 | 綠色 |
| `chordal` | 基於弦長參數化，適合不均勻間距的點 | 藍色 |

### 實作方式

```javascript
// 建立曲線
const curve = new THREE.CatmullRomCurve3(positions);
curve.curveType = 'catmullrom'; // 或 'centripetal', 'chordal'
curve.tension = 0.5; // 僅對 catmullrom 類型有效，範圍 0-1

// 取得曲線上的點用於渲染
const points = curve.getPoints(ARC_SEGMENTS); // 例如 200 段
```

### 曲線幾何更新

```javascript
function updateSplineOutline() {
  for (const k in splines) {
    const spline = splines[k];
    const splineMesh = spline.mesh;
    const position = splineMesh.geometry.attributes.position;

    for (let i = 0; i < ARC_SEGMENTS; i++) {
      const t = i / (ARC_SEGMENTS - 1);
      spline.getPoint(t, point);
      position.setXYZ(i, point.x, point.y, point.z);
    }
    position.needsUpdate = true;
  }
}
```

---

## 3. TransformControls 整合

### Decision: 使用 TransformControls addon 實現拖曳功能

**Rationale**: Three.js 官方提供的 TransformControls 支援平移、旋轉、縮放操作，適合控制點編輯。

### 實作方式

```javascript
import { TransformControls } from 'three/addons/controls/TransformControls.js';

// 建立 TransformControls
const transformControl = new TransformControls(camera, renderer.domElement);
transformControl.addEventListener('change', render);
transformControl.addEventListener('dragging-changed', function(event) {
  // 拖曳時停用 OrbitControls
  controls.enabled = !event.value;
});
scene.add(transformControl.getHelper());

// 綁定到物件
transformControl.attach(selectedObject);

// 解除綁定
transformControl.detach();
```

### 物件變更監聽

```javascript
transformControl.addEventListener('objectChange', function() {
  updateSplineOutline();
});
```

---

## 4. OrbitControls 與 TransformControls 協調

### Decision: 透過 dragging-changed 事件協調兩個控制器

**Rationale**: 當拖曳 TransformControls 時，必須停用 OrbitControls 以避免衝突。

### 實作方式

```javascript
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const controls = new OrbitControls(camera, renderer.domElement);
controls.damping = 0.2;
controls.addEventListener('change', render);

// 在 TransformControls 拖曳時停用
transformControl.addEventListener('dragging-changed', function(event) {
  controls.enabled = !event.value;
});
```

---

## 5. lil-gui 面板配置

### Decision: 使用 lil-gui 建立控制面板

**Rationale**: lil-gui 是 Three.js 官方範例採用的輕量級 GUI 函式庫，簡單易用。

### 實作方式

```javascript
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';

const params = {
  uniform: true,
  tension: 0.5,
  centripetal: true,
  chordal: true,
  addPoint: addPoint,
  removePoint: removePoint,
  exportSpline: exportSpline
};

const gui = new GUI();
gui.add(params, 'uniform').onChange(render);
gui.add(params, 'tension', 0, 1).step(0.01).onChange(function(value) {
  splines.uniform.tension = value;
  updateSplineOutline();
  render();
});
gui.add(params, 'centripetal').onChange(render);
gui.add(params, 'chordal').onChange(render);
gui.add(params, 'addPoint');
gui.add(params, 'removePoint');
gui.add(params, 'exportSpline');
gui.open();
```

---

## 6. Raycaster 選取機制

### Decision: 使用 Raycaster 偵測滑鼠點擊選取控制點

**Rationale**: Raycaster 是 Three.js 標準的 3D 物件選取方式。

### 實作方式

```javascript
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();

function onPointerDown(event) {
  onDownPosition.x = event.clientX;
  onDownPosition.y = event.clientY;
}

function onPointerUp(event) {
  onUpPosition.x = event.clientX;
  onUpPosition.y = event.clientY;

  // 只有在點擊（非拖曳）時才選取
  if (onDownPosition.distanceTo(onUpPosition) === 0) {
    pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
    pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;

    raycaster.setFromCamera(pointer, camera);
    const intersects = raycaster.intersectObjects(splineHelperObjects, false);

    if (intersects.length > 0) {
      const object = intersects[0].object;
      if (object !== transformControl.object) {
        transformControl.attach(object);
      }
    } else {
      transformControl.detach();
    }
    render();
  }
}
```

---

## 7. 控制點管理

### Decision: 使用陣列管理控制點物件和位置

**Rationale**: 便於新增、移除和遍歷控制點。

### 實作方式

```javascript
const splineHelperObjects = [];
const positions = [];
let splinePointsLength = 4;
const geometry = new THREE.BoxGeometry(20, 20, 20);

function addSplineObject(position) {
  const material = new THREE.MeshLambertMaterial({ 
    color: Math.random() * 0xffffff 
  });
  const object = new THREE.Mesh(geometry, material);
  
  if (position) {
    object.position.copy(position);
  } else {
    object.position.x = Math.random() * 1000 - 500;
    object.position.y = Math.random() * 600;
    object.position.z = Math.random() * 800 - 400;
  }
  
  object.castShadow = true;
  object.receiveShadow = true;
  scene.add(object);
  splineHelperObjects.push(object);
  return object;
}

function addPoint() {
  splinePointsLength++;
  positions.push(addSplineObject().position);
  updateSplineOutline();
  render();
}

function removePoint() {
  if (splinePointsLength <= 4) return;
  
  const point = splineHelperObjects.pop();
  splinePointsLength--;
  positions.pop();
  
  if (transformControl.object === point) {
    transformControl.detach();
  }
  
  scene.remove(point);
  updateSplineOutline();
  render();
}
```

---

## 8. 曲線匯出功能

### Decision: 將控制點座標以 JavaScript 陣列格式輸出到主控台

**Rationale**: 便於開發者複製貼上到其他專案中使用。

### 實作方式

```javascript
function exportSpline() {
  const strplace = [];
  for (let i = 0; i < splinePointsLength; i++) {
    const p = splineHelperObjects[i].position;
    strplace.push(`new THREE.Vector3(${p.x}, ${p.y}, ${p.z})`);
  }
  console.log(strplace.join(',\n'));
  
  // 也可以輸出 load() 函式格式
  const code = '[' + strplace.join(',\n\t') + ']';
  prompt('Copy to clipboard: Ctrl+C, Enter', code);
}
```

---

## 9. 場景設置

### Decision: 建立包含地面、網格、光源和陰影的完整場景

**Rationale**: 提供視覺參考和美觀的 3D 環境。

### 實作方式

```javascript
function init() {
  // 場景
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0xf0f0f0);

  // 相機
  camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 1, 10000);
  camera.position.set(0, 250, 1000);
  scene.add(camera);

  // 環境光
  scene.add(new THREE.AmbientLight(0xf0f0f0, 3));

  // 聚光燈（帶陰影）
  const light = new THREE.SpotLight(0xffffff, 4.5);
  light.position.set(0, 1500, 200);
  light.angle = Math.PI * 0.2;
  light.decay = 0;
  light.castShadow = true;
  light.shadow.camera.near = 200;
  light.shadow.camera.far = 2000;
  light.shadow.bias = -0.000222;
  light.shadow.mapSize.width = 1024;
  light.shadow.mapSize.height = 1024;
  scene.add(light);

  // 地面
  const planeGeometry = new THREE.PlaneGeometry(2000, 2000);
  planeGeometry.rotateX(-Math.PI / 2);
  const planeMaterial = new THREE.ShadowMaterial({ color: 0x000000, opacity: 0.2 });
  const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  plane.position.y = -200;
  plane.receiveShadow = true;
  scene.add(plane);

  // 網格
  const helper = new THREE.GridHelper(2000, 100);
  helper.position.y = -199;
  helper.material.opacity = 0.25;
  helper.material.transparent = true;
  scene.add(helper);

  // 渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.shadowMap.enabled = true;
  container.appendChild(renderer.domElement);
}
```

---

## 10. 視窗大小調整

### Decision: 監聽 resize 事件更新相機和渲染器

**Rationale**: 確保場景在視窗大小改變時正確調整。

### 實作方式

```javascript
window.addEventListener('resize', onWindowResize);

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
  render();
}
```

---

## 研究結論

所有技術細節已釐清，可進入 Phase 1 設計階段。關鍵技術包括：

1. **CatmullRomCurve3**: 三種曲線類型 + tension 參數
2. **TransformControls**: 拖曳控制點 + 與 OrbitControls 協調
3. **Raycaster**: 點擊選取控制點
4. **lil-gui**: 控制面板配置
5. **BufferGeometry**: 動態更新曲線幾何

預計實作檔案：`examples/webgl-spline-editor/index.html`
