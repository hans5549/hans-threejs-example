# Quickstart: WebGL 文字描邊效果

**Feature Branch**: `018-text-stroke`  
**Created**: 2025-12-04

## 功能概述

此範例展示如何使用 Three.js 建立具有描邊效果的 3D 文字，同時支援多種書寫方向（左到右、右到左、上到下）。

## 核心技術

- **Font.generateShapes**: 從字體產生文字形狀
- **ShapeGeometry**: 建立填充幾何體
- **SVGLoader.pointsToStroke**: 建立描邊幾何體
- **fflate**: 解壓縮 zip 格式字體檔案

## 快速範例

### 1. 基本設定

```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
  }
}
</script>
```

### 2. 匯入模組

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { SVGLoader } from 'three/addons/loaders/SVGLoader.js';
import { Font } from 'three/addons/loaders/FontLoader.js';
import { unzipSync, strFromU8 } from 'three/addons/libs/fflate.module.js';
```

### 3. 載入字體

```javascript
new THREE.FileLoader()
  .setResponseType('arraybuffer')
  .load('fonts/MPLUSRounded1c/MPLUSRounded1c-Regular.typeface.json.zip', (data) => {
    const zip = unzipSync(new Uint8Array(data));
    const json = strFromU8(zip['MPLUSRounded1c-Regular.typeface.json']);
    const font = new Font(JSON.parse(json));
    
    // 建立文字...
  });
```

### 4. 建立描邊文字

```javascript
function generateStrokeText(font, material, message, size, direction = 'ltr') {
  // 產生文字形狀
  const shapes = font.generateShapes(message, size, direction);
  
  // 建立填充幾何體
  const geometry = new THREE.ShapeGeometry(shapes);
  geometry.computeBoundingBox();
  const xMid = -0.5 * (geometry.boundingBox.max.x - geometry.boundingBox.min.x);
  geometry.translate(xMid, 0, 0);
  
  // 建立填充網格
  const strokeText = new THREE.Group();
  const text = new THREE.Mesh(geometry, material.lite);
  text.position.z = -150;
  strokeText.add(text);
  
  // 提取孔洞
  const holeShapes = [];
  for (const shape of shapes) {
    if (shape.holes?.length > 0) {
      holeShapes.push(...shape.holes);
    }
  }
  shapes.push(...holeShapes);
  
  // 建立描邊
  const style = SVGLoader.getStrokeStyle(5, material.color.getStyle());
  for (const shape of shapes) {
    const points = shape.getPoints();
    const strokeGeometry = SVGLoader.pointsToStroke(points, style);
    strokeGeometry.translate(xMid, 0, 0);
    const strokeMesh = new THREE.Mesh(strokeGeometry, material.dark);
    strokeText.add(strokeMesh);
  }
  
  return strokeText;
}
```

### 5. 使用範例

```javascript
// 定義材質
const color = new THREE.Color(0x006699);
const material = {
  dark: new THREE.MeshBasicMaterial({ color, side: THREE.DoubleSide }),
  lite: new THREE.MeshBasicMaterial({ 
    color, 
    transparent: true, 
    opacity: 0.4, 
    side: THREE.DoubleSide 
  }),
  color
};

// 建立不同方向的文字
const english = generateStrokeText(font, material, 'Three.js\nStroke text.', 80, 'ltr');
const hebrew = generateStrokeText(font, material, 'טקסט קו', 80, 'rtl');
const chinese = generateStrokeText(font, material, '文字描邊', 80, 'tb');

// 定位
english.position.set(-100, 0, 0);
hebrew.position.set(-100, -300, 0);
chinese.position.set(300, -300, 0);

// 加入場景
scene.add(english, hebrew, chinese);
```

## 檔案結構

```
examples/
└── webgl-geometry-text-stroke/
    └── index.html          # 完整範例

fonts/
└── MPLUSRounded1c/
    └── MPLUSRounded1c-Regular.typeface.json.zip  # 字體檔案
```

## 執行方式

1. 確保字體檔案已放置在正確位置
2. 使用本地伺服器開啟 `index.html`（例如 `npx serve`）
3. 在瀏覽器中檢視 3D 文字描邊效果
4. 使用滑鼠拖曳旋轉視角、滾輪縮放

## 常見問題

### Q: 字體載入失敗？
A: 確認字體檔案路徑正確，且使用本地伺服器而非直接開啟檔案。

### Q: 文字方向不正確？
A: 確認使用 Three.js r166+ 版本，舊版本可能不支援 direction 參數。

### Q: 描邊線條過粗或過細？
A: 調整 `SVGLoader.getStrokeStyle(width, color)` 的第一個參數。
