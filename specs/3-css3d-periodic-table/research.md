# Research: CSS3D 週期表

**Feature**: 3-css3d-periodic-table  
**Date**: 2025-11-28  
**Purpose**: 解決 Technical Context 中的技術問題並研究最佳實踐

---

## 1. CSS3DRenderer 技術研究

### Decision: 使用 Three.js CSS3DRenderer

**Rationale**: 
- CSS3DRenderer 允許將標準 HTML DOM 元素渲染到 3D 空間中
- 相比 WebGL 文字渲染，CSS 文字更清晰、可選取、支援完整 CSS 樣式
- 非常適合需要豐富文字顯示的 3D UI 場景（如週期表）

**Alternatives Considered**:
- WebGL TextGeometry: 文字品質較差，不支援 CSS 樣式
- HTML Canvas 貼圖: 需要手動管理文字渲染，複雜度高
- CSS3DRenderer ✅: 原生 DOM 元素，最佳文字品質和互動性

### 使用方式

```javascript
import { CSS3DRenderer, CSS3DObject } from 'three/addons/renderers/CSS3DRenderer.js';

// 建立 renderer
const renderer = new CSS3DRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);

// 將 DOM 元素轉換為 3D 物件
const element = document.createElement('div');
const object = new CSS3DObject(element);
scene.add(object);
```

---

## 2. Tween 動畫函式庫研究

### Decision: 使用 @tweenjs/tween.js

**Rationale**:
- Three.js 官方範例使用此函式庫
- 輕量級、API 簡潔
- 支援多種緩動函式 (Easing Functions)
- 可同時處理多個屬性動畫

**Alternatives Considered**:
- GSAP: 功能強大但檔案較大，對簡單動畫過於複雜
- anime.js: 類似 GSAP，對此需求過度
- 原生 requestAnimationFrame: 需手動實作緩動計算
- @tweenjs/tween.js ✅: 輕量、官方範例使用、完美契合需求

### 使用方式

```javascript
import TWEEN from 'three/addons/libs/tween.module.js';

// 建立動畫
new TWEEN.Tween(object.position)
    .to({ x: target.x, y: target.y, z: target.z }, 2000)
    .easing(TWEEN.Easing.Exponential.InOut)
    .start();

// 在動畫迴圈中更新
function animate() {
    requestAnimationFrame(animate);
    TWEEN.update();
}
```

---

## 3. TrackballControls 互動控制研究

### Decision: 使用 TrackballControls

**Rationale**:
- 提供 360 度全方位旋轉控制
- 支援滑鼠拖曳旋轉、滾輪縮放
- 可設定距離限制 (minDistance, maxDistance)
- 適合需要自由視角探索的 3D 場景

**Alternatives Considered**:
- OrbitControls: 限制垂直旋轉角度，不適合此場景
- FlyControls: 適合第一人稱視角，非此需求
- TrackballControls ✅: 最適合 3D 物件展示的自由旋轉

### 使用方式

```javascript
import { TrackballControls } from 'three/addons/controls/TrackballControls.js';

const controls = new TrackballControls(camera, renderer.domElement);
controls.minDistance = 500;
controls.maxDistance = 6000;
controls.addEventListener('change', render);
```

---

## 4. 四種視圖佈局演算法

### 4.1 Table Layout (週期表)

**Decision**: 基於欄列位置的二維平面佈局

```javascript
// 每個元素有 column (1-18) 和 row (1-10) 資訊
object.position.x = (column * 140) - 1330;  // 水平間距 140px
object.position.y = -(row * 180) + 990;     // 垂直間距 180px
object.position.z = 0;                       // 平面
```

### 4.2 Sphere Layout (球形)

**Decision**: 使用球面座標系統均勻分布

```javascript
const phi = Math.acos(-1 + (2 * i) / total);           // 極角
const theta = Math.sqrt(total * Math.PI) * phi;         // 方位角
object.position.setFromSphericalCoords(800, phi, theta);
object.lookAt(center);  // 面向中心
```

### 4.3 Helix Layout (螺旋)

**Decision**: 使用圓柱座標系統建立螺旋

```javascript
const theta = i * 0.175 + Math.PI;  // 角度遞增
const y = -(i * 8) + 450;           // 垂直位置遞減
object.position.setFromCylindricalCoords(900, theta, y);
object.lookAt(externalPoint);       // 面向外側
```

### 4.4 Grid Layout (網格)

**Decision**: 三維網格排列 (5×5×5 結構)

```javascript
object.position.x = ((i % 5) * 400) - 800;              // X: 0-4
object.position.y = (-(Math.floor(i / 5) % 5) * 400) + 800;  // Y: 0-4
object.position.z = (Math.floor(i / 25)) * 1000 - 2000;      // Z: 0-4
```

---

## 5. 化學元素資料結構

### Decision: 使用扁平陣列儲存元素資料

**Rationale**:
- 符合原始範例的資料格式
- 減少物件建立開銷
- 容易遍歷處理

**Data Format**:
```javascript
const table = [
    // 符號, 名稱, 原子量, 欄位(1-18), 列(1-10)
    'H', 'Hydrogen', '1.00794', 1, 1,
    'He', 'Helium', '4.002602', 18, 1,
    // ... 共 118 個元素 × 5 = 590 個值
];
```

---

## 6. CDN 載入策略

### Decision: 使用 unpkg CDN + ES Module Import Map

**Rationale**:
- 與專案現有範例一致 (參考 interactive-raycasting-points)
- 無需本地安裝 node_modules
- 支援 ES Module 動態載入

**Import Map Configuration**:
```html
<script type="importmap">
{
    "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
    }
}
</script>
```

---

## Summary

| 技術決策 | 選擇 | 理由 |
|----------|------|------|
| 3D 渲染 | CSS3DRenderer | DOM 元素高品質文字 |
| 動畫函式庫 | @tweenjs/tween.js | 輕量、官方範例使用 |
| 互動控制 | TrackballControls | 360 度自由旋轉 |
| 資料格式 | 扁平陣列 | 效能、相容原始範例 |
| CDN | unpkg | 專案一致性 |
