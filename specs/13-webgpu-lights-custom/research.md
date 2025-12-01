# Research: WebGPU Custom Lighting Model

**Feature**: 13-webgpu-lights-custom  
**Date**: 2025-12-01  
**Purpose**: 解決技術背景中的不確定項目並記錄技術決策

## Research Tasks

### 1. Three.js WebGPU 渲染器配置

**問題**: 如何正確初始化 WebGPURenderer？

**調查結果**:
- WebGPURenderer 是 Three.js r150+ 提供的 WebGPU 原生渲染器
- 需從 `three.webgpu.js` 匯入，使用 `import * as THREE from 'three/webgpu'`
- 渲染器初始化是同步的，但需要 WebGPU adapter 初始化
- WebGPU 功能偵測：檢查 `navigator.gpu` 是否存在

**決策**: 使用 Three.js CDN 的 WebGPU build（unpkg 或 jsDelivr），使用 import map 配置

**Import Map 配置**:
```json
{
  "imports": {
    "three": "https://unpkg.com/three@0.170.0/build/three.webgpu.js",
    "three/webgpu": "https://unpkg.com/three@0.170.0/build/three.webgpu.js",
    "three/tsl": "https://unpkg.com/three@0.170.0/build/three.tsl.js",
    "three/addons/": "https://unpkg.com/three@0.170.0/examples/jsm/"
  }
}
```

---

### 2. TSL (Three Shading Language) 與自訂光照模型

**問題**: 如何使用 TSL 實作自訂光照模型？

**調查結果**:
- `THREE.LightingModel` 是光照模型的基礎類別
- 透過覆寫 `direct()` 方法定義直接光照計算
- `direct()` 接收 `{ lightColor, reflectedLight }` 參數
- `reflectedLight.directDiffuse` 是漫反射累加器

**決策**: 建立繼承 `THREE.LightingModel` 的 `CustomLightingModel` 類別

**關鍵程式碼模式**:
```javascript
class CustomLightingModel extends THREE.LightingModel {
    direct( { lightColor, reflectedLight } ) {
        // 直接將光源顏色加到漫反射
        reflectedLight.directDiffuse.addAssign( lightColor );
    }
}
```

---

### 3. 選擇性光源節點 (Selective Lights)

**問題**: 如何讓特定光源只影響特定材質？

**調查結果**:
- TSL 的 `lights()` 函式可接受光源陣列作為參數
- 語法：`lights([light1, light2, light3])` 建立選擇性光源節點
- 使用 `.context({ lightingModel })` 將自訂光照模型應用到光源節點
- 將結果設定為材質的 `lightsNode` 屬性

**決策**: 使用 `lights()` 搭配 `.context()` 實現選擇性光照

**關鍵程式碼模式**:
```javascript
import { lights } from 'three/tsl';

const allLightsNode = lights( [ light1, light2, light3 ] );
const lightingModel = new CustomLightingModel();
const lightingModelContext = allLightsNode.context( { lightingModel } );

materialPoints.lightsNode = lightingModelContext;
```

---

### 4. 光源小球自發光材質

**問題**: 如何讓光源指示小球不受其他光源影響？

**調查結果**:
- `NodeMaterial` 的 `lightsNode` 屬性可設為空的 `lights()` 節點
- 空的 `lights()` 調用表示忽略場景中所有光源
- 使用 `colorNode` 設定純色

**決策**: 光源小球使用 `NodeMaterial` 並設定 `lightsNode = lights()`

**關鍵程式碼模式**:
```javascript
import { color, lights } from 'three/tsl';

const material = new THREE.NodeMaterial();
material.colorNode = color( hexColor );
material.lightsNode = lights(); // 空陣列 = 忽略所有場景光源
```

---

### 5. PointsNodeMaterial 使用

**問題**: 點雲應使用什麼材質？

**調查結果**:
- `PointsNodeMaterial` 是支援 TSL 的點雲材質
- 繼承自 `NodeMaterial`，支援 `lightsNode` 屬性
- 預設粒子大小約 1 像素（符合需求）
- 預設顏色為白色（符合需求）

**決策**: 使用 `PointsNodeMaterial` 並設定自訂 `lightsNode`

---

### 6. 點雲效能考量

**問題**: 50 萬粒子是否能達到 60 FPS？

**調查結果**:
- Three.js Points 使用 GL_POINTS 渲染，效能優異
- 50 萬點在現代 GPU 上應可達 60 FPS
- WebGPU 渲染比 WebGL 更高效
- 自訂光照模型為簡單加法，計算負擔低

**決策**: 維持 50 萬粒子，若效能不足再降低

**備選方案**:
- 降低粒子數量至 25 萬
- 使用 InstancedMesh 代替 Points（更複雜但可控制大小）

---

### 7. OrbitControls 整合

**問題**: 如何整合 OrbitControls？

**調查結果**:
- OrbitControls 從 `three/addons/controls/OrbitControls.js` 匯入
- 建構時傳入 camera 和 renderer.domElement
- 支援 `minDistance` 和 `maxDistance` 限制縮放範圍
- 預設支援觸控（滿足需求）

**決策**: 使用標準 OrbitControls 配置

**關鍵程式碼模式**:
```javascript
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const controls = new OrbitControls( camera, renderer.domElement );
controls.minDistance = 0;
controls.maxDistance = 4;
```

---

### 8. WebGPU 不支援時的 Fallback

**問題**: 如何處理 WebGPU 不支援的情況？

**調查結果**:
- 檢測：`if (!navigator.gpu)` 表示不支援
- 需要準備靜態 fallback 圖片
- 顯示說明文字告知使用者

**決策**: 使用 CSS 隱藏 canvas，顯示 fallback 圖片和說明

**備選方案考慮後拒絕**:
- 降級到 WebGL：失去自訂光照模型的核心價值
- 使用預渲染影片：檔案過大，不適合

---

## Summary

所有技術疑問已解決，未發現阻斷問題。可進入 Phase 1 設計階段。

| 項目 | 決策 |
|------|------|
| 渲染器 | WebGPURenderer + CDN import map |
| 光照模型 | 繼承 THREE.LightingModel |
| 選擇性光源 | TSL lights() + context() |
| 光源指示 | NodeMaterial + 空 lightsNode |
| 點雲材質 | PointsNodeMaterial |
| 粒子數量 | 50 萬（依效能調整） |
| 相機控制 | OrbitControls |
| Fallback | 靜態圖片 + 說明文字 |
