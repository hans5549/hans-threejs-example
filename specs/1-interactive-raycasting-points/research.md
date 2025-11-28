# Research: Three.js 互動式點雲 Raycasting 範例

**Feature**: `1-interactive-raycasting-points`  
**Date**: 2025-11-28  
**Status**: Complete

## 研究摘要

本研究文件整理了實作 Three.js 互動式點雲 Raycasting 範例所需的技術細節，基於 Three.js 官方範例 `webgl_interactive_raycasting_points` 進行分析。

---

## 1. Three.js CDN 載入方式

### Decision: 使用 Import Map + ES Module

**Rationale**: 這是 Three.js r160+ 推薦的現代載入方式，無需建置工具，直接在瀏覽器中使用 ES Module。

**Alternatives considered**:
- UMD 全域變數方式 - 已過時，不支援 tree-shaking
- npm + bundler - 對於單一展示範例過於複雜

### 實作方式

```html
<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
  }
}
</script>

<script type="module">
import * as THREE from 'three';
import Stats from 'three/addons/libs/stats.module.js';
// ... 程式碼
</script>
```

---

## 2. BufferGeometry 點雲建立

### Decision: 使用 Float32Array + BufferAttribute

**Rationale**: BufferGeometry 是 Three.js 中效能最佳的幾何建立方式，直接使用 TypedArray 減少記憶體開銷和 GPU 傳輸時間。

### 實作方式

```javascript
function generatePointCloudGeometry(color, width, length) {
  const geometry = new THREE.BufferGeometry();
  const numPoints = width * length;

  const positions = new Float32Array(numPoints * 3);
  const colors = new Float32Array(numPoints * 3);

  let k = 0;
  for (let i = 0; i < width; i++) {
    for (let j = 0; j < length; j++) {
      const u = i / width;
      const v = j / length;
      
      // 位置計算 (波浪形狀)
      const x = u - 0.5;
      const y = (Math.cos(u * Math.PI * 4) + Math.sin(v * Math.PI * 8)) / 20;
      const z = v - 0.5;

      positions[3 * k] = x;
      positions[3 * k + 1] = y;
      positions[3 * k + 2] = z;

      // 顏色計算 (基於 Y 軸強度)
      const intensity = (y + 0.1) * 5;
      colors[3 * k] = color.r * intensity;
      colors[3 * k + 1] = color.g * intensity;
      colors[3 * k + 2] = color.b * intensity;

      k++;
    }
  }

  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));
  geometry.computeBoundingBox();

  return geometry;
}
```

### 三種點雲類型

| 類型 | 說明 | 實作差異 |
|------|------|----------|
| Standard | 標準點雲 | 僅設定 position 和 color attributes |
| Indexed | 帶索引點雲 | 額外呼叫 `geometry.setIndex()` |
| Indexed + Offset | 帶分組偏移 | 額外呼叫 `geometry.addGroup(0, indices.length)` |

---

## 3. Raycaster Points 偵測

### Decision: 使用 Raycaster.params.Points.threshold

**Rationale**: Three.js 的 Raycaster 內建對 Points 物件的支援，透過 threshold 參數控制偵測靈敏度。

### 實作方式

```javascript
const raycaster = new THREE.Raycaster();
raycaster.params.Points.threshold = 0.1; // 偵測閾值

function onPointerMove(event) {
  pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
  pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;
}

function render() {
  raycaster.setFromCamera(pointer, camera);
  
  const intersections = raycaster.intersectObjects(pointclouds, false);
  const intersection = intersections.length > 0 ? intersections[0] : null;
  
  if (intersection !== null) {
    // 在 intersection.point 位置顯示標記
  }
}
```

### 重要參數

| 參數 | 值 | 說明 |
|------|-----|------|
| `threshold` | 0.1 | 射線與點的最大距離，值越大越容易偵測到 |
| `intersections[0].point` | Vector3 | 交集點的世界座標 |
| `intersections[0].index` | number | 被選中點在 BufferGeometry 中的索引 |

---

## 4. Stats.js 整合

### Decision: 使用 Three.js 內建的 Stats 模組

**Rationale**: Stats.js 已包含在 Three.js addons 中，無需額外載入，提供 FPS、MS、MB 三種監控模式。

### 實作方式

```javascript
import Stats from 'three/addons/libs/stats.module.js';

let stats;

function init() {
  stats = new Stats();
  container.appendChild(stats.dom);
}

function animate() {
  render();
  stats.update(); // 每幀更新
}
```

---

## 5. 攝影機旋轉矩陣

### Decision: 使用 Matrix4.makeRotationY + applyMatrix4

**Rationale**: 透過矩陣運算旋轉攝影機，比直接修改 position 更精確，保持攝影機始終看向場景中心。

### 實作方式

```javascript
const rotateY = new THREE.Matrix4().makeRotationY(0.005); // 每幀旋轉角度

function render() {
  camera.applyMatrix4(rotateY);
  camera.updateMatrixWorld();
  
  // ... 其餘渲染邏輯
  renderer.render(scene, camera);
}
```

---

## 6. 球體標記物件池

### Decision: 預先建立 40 個球體並重複使用

**Rationale**: 避免頻繁建立和銷毀物件造成的效能問題和記憶體碎片。

### 實作方式

```javascript
const spheres = [];
let spheresIndex = 0;

function init() {
  const sphereGeometry = new THREE.SphereGeometry(0.1, 32, 32);
  const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });

  for (let i = 0; i < 40; i++) {
    const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
    scene.add(sphere);
    spheres.push(sphere);
  }
}

function render() {
  // 使用下一個球體
  if (intersection !== null) {
    spheres[spheresIndex].position.copy(intersection.point);
    spheres[spheresIndex].scale.set(1, 1, 1);
    spheresIndex = (spheresIndex + 1) % spheres.length;
  }

  // 所有球體逐漸縮小
  for (let i = 0; i < spheres.length; i++) {
    spheres[i].scale.multiplyScalar(0.98);
    spheres[i].scale.clampScalar(0.01, 1);
  }
}
```

---

## 7. 時間控制 (Toggle 機制)

### Decision: 使用 Clock.getDelta() 控制標記生成頻率

**Rationale**: 避免每幀都產生新標記，透過時間間隔 (約 20ms) 控制更新頻率。

### 實作方式

```javascript
let toggle = 0;
const clock = new THREE.Clock();

function render() {
  // ... raycasting 邏輯

  if (toggle > 0.02 && intersection !== null) {
    // 產生新標記
    toggle = 0;
  }

  toggle += clock.getDelta();
}
```

---

## 研究結論

所有技術細節已確認，無需額外澄清。可直接進入 Phase 1 設計階段。

| 研究項目 | 狀態 | 備註 |
|----------|------|------|
| CDN 載入 | ✅ Complete | Import Map + ES Module |
| BufferGeometry | ✅ Complete | Float32Array + BufferAttribute |
| Raycaster | ✅ Complete | threshold = 0.1 |
| Stats.js | ✅ Complete | three/addons/libs/stats.module.js |
| 攝影機旋轉 | ✅ Complete | Matrix4.makeRotationY |
| 物件池 | ✅ Complete | 40 個球體標記 |
| 時間控制 | ✅ Complete | Clock.getDelta() |
