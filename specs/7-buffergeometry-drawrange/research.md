# Research: BufferGeometry DrawRange 互動式粒子網絡

**Date**: 2025-11-28  
**Purpose**: 解決 Technical Context 中的 NEEDS CLARIFICATION 項目並確立最佳實踐

---

## 1. BufferGeometry 動態更新最佳實踐

### 決策：使用 DynamicDrawUsage 配合 needsUpdate

**研究發現**:

Three.js BufferGeometry 提供兩種主要的使用模式：
- `THREE.StaticDrawUsage`：資料不常更新，GPU 會優化為靜態緩衝區
- `THREE.DynamicDrawUsage`：資料頻繁更新，GPU 會優化為動態緩衝區

**選擇 DynamicDrawUsage 的理由**:
1. 粒子位置每幀都會更新（移動、反彈）
2. 連線數量和頂點每幀都會變化
3. Three.js 官方範例使用此模式處理動態幾何

**needsUpdate 使用時機**:
```javascript
// 位置資料更新後必須設定
particles.geometry.attributes.position.needsUpdate = true;
linesMesh.geometry.attributes.position.needsUpdate = true;
linesMesh.geometry.attributes.color.needsUpdate = true;
```

**setDrawRange 最佳實踐**:
```javascript
// 控制實際渲染的元素數量，避免渲染未使用的緩衝區空間
geometry.setDrawRange(0, actualVertexCount);
```

**替代方案評估**:
- `BufferGeometry.dispose()` + 重建：效能差，每幀建立/銷毀物件會造成 GC 壓力
- `InstancedBufferGeometry`：適合相同幾何的重複實例，不適用於連線場景

---

## 2. 粒子間距離計算優化

### 決策：採用簡單 O(n²) 演算法

**研究發現**:

對於 1000 個粒子的最壞情況：
- 距離計算次數：n(n-1)/2 = 499,500 次
- 每次計算：3 次減法 + 3 次乘法 + 1 次加法 + 1 次開平方
- 現代 JavaScript 引擎可在 <5ms 內完成

**效能測試結論**:
| 粒子數 | 預估計算時間 | FPS 影響 |
|--------|-------------|---------|
| 200 | <1ms | 可忽略 |
| 500 | ~2ms | 可忽略 |
| 1000 | ~5ms | 可接受（60FPS 有 16ms 預算） |

**選擇簡單演算法的理由**:
1. 1000 粒子上限足夠滿足效能需求
2. 空間分割（Octree/BVH）會增加程式碼複雜度
3. Three.js 官方範例使用相同方法，驗證其可行性
4. 避免過度工程化

**潛在優化（如未來需要）**:
- 使用平方距離比較，避免 sqrt 運算
- Web Workers 進行背景計算
- GPU Compute Shader（WebGPU）

**實作技巧**:
```javascript
// 使用平方距離避免 sqrt（比較時）
const distSq = dx * dx + dy * dy + dz * dz;
const minDistSq = minDistance * minDistance;
if (distSq < minDistSq) {
  // 只有在需要 alpha 時才計算實際距離
  const dist = Math.sqrt(distSq);
  const alpha = 1.0 - dist / minDistance;
}
```

---

## 3. lil-gui 整合模式

### 決策：使用現有專案的 CDN 載入模式

**研究發現**:

檢視現有專案範例（如 `unreal-bloom-postprocessing/`），確認使用模式：

```javascript
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';
```

**GUI 控制器配置最佳實踐**:
```javascript
const effectController = {
  showDots: true,
  showLines: true,
  minDistance: 150,
  limitConnections: false,
  maxConnections: 20,
  particleCount: 500
};

const gui = new GUI();
gui.add(effectController, 'showDots').onChange(value => {
  pointCloud.visible = value;
});
gui.add(effectController, 'particleCount', 0, maxParticleCount, 1)
   .onChange(value => {
     particleCount = value;
     particles.setDrawRange(0, particleCount);
   });
```

**選擇此模式的理由**:
1. 與專案現有範例保持一致
2. 無需額外的套件管理
3. CDN 載入確保版本相容

---

## 4. Three.js 版本相容性

### 決策：使用 Three.js r160

**研究發現**:

檢視現有專案使用的版本：
```json
{
  "imports": {
    "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
  }
}
```

**API 確認**:
- `BufferGeometry.setDrawRange()` - 自 r73 起可用
- `BufferAttribute.setUsage()` - 自 r110 起可用
- `OrbitControls` - 穩定 API
- `Stats` - 穩定 API
- `GUI (lil-gui)` - 穩定 API

---

## 5. 視覺效果配置

### 決策：使用 AdditiveBlending 和透明材質

**研究發現**:

Three.js 官方範例使用的視覺效果配置：

**粒子材質**:
```javascript
const pMaterial = new THREE.PointsMaterial({
  color: 0xFFFFFF,
  size: 3,
  blending: THREE.AdditiveBlending,
  transparent: true,
  sizeAttenuation: false  // 粒子大小不隨距離變化
});
```

**連線材質**:
```javascript
const material = new THREE.LineBasicMaterial({
  vertexColors: true,
  blending: THREE.AdditiveBlending,
  transparent: true
});
```

**選擇此配置的理由**:
1. AdditiveBlending 產生發光效果，適合暗色背景
2. 透明度支援距離衰減效果
3. vertexColors 支援每頂點獨立的透明度

---

## 研究結論

所有技術決策已確定，無需進一步澄清：

| 項目 | 決策 | 信心度 |
|------|------|--------|
| 緩衝區使用模式 | DynamicDrawUsage | 高 |
| 距離計算演算法 | O(n²) 簡單迴圈 | 高 |
| GUI 框架 | lil-gui via CDN | 高 |
| Three.js 版本 | r160.0 | 高 |
| 視覺效果 | AdditiveBlending | 高 |

**可進入 Phase 1 設計階段**
