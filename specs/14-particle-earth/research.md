# Research: Particle Earth Sphere

**Feature**: 14-particle-earth  
**Date**: 2025-12-01  
**Purpose**: 解決 Technical Context 中的技術問題並記錄研究結果

## Research Tasks

### 1. 斐波那契球面分布演算法

**Task**: 研究如何在球面上均勻分布粒子

**Decision**: 使用黃金角（Golden Angle）演算法

**Rationale**: 
- 黃金角 φ = π(3 - √5) ≈ 2.399963 弧度
- 此方法產生的點分布視覺上最均勻，避免極點聚集問題
- 計算複雜度 O(n)，適合即時生成

**Implementation**:
```javascript
function generateFibonacciSphere(numPoints, radius) {
    const positions = new Float32Array(numPoints * 3);
    const phi = Math.PI * (3 - Math.sqrt(5)); // 黃金角
    
    for (let i = 0; i < numPoints; i++) {
        const y = 1 - (i / (numPoints - 1)) * 2; // y: 1 to -1
        const radiusAtY = Math.sqrt(1 - y * y);
        const theta = phi * i;
        
        const x = Math.cos(theta) * radiusAtY;
        const z = Math.sin(theta) * radiusAtY;
        
        positions[i * 3] = x * radius;
        positions[i * 3 + 1] = y * radius;
        positions[i * 3 + 2] = z * radius;
    }
    
    return positions;
}
```

**Alternatives Considered**:
- 經緯度網格：極點過密，視覺不均勻
- 隨機分布：不夠均勻，有明顯空洞

---

### 2. 球面 UV 座標計算

**Task**: 將球面座標轉換為紋理 UV 座標

**Decision**: 在 Vertex Shader 中使用等距柱狀投影公式

**Rationale**:
- 等距柱狀投影（Equirectangular）是地球紋理的標準格式
- NASA Blue Marble 圖片使用此投影
- 計算簡單，只需 atan2 和 asin

**Implementation (GLSL)**:
```glsl
// 從正規化位置計算 UV
vec2 calculateUV(vec3 position) {
    vec3 normalized = normalize(position);
    float u = 0.5 + atan(normalized.z, normalized.x) / (2.0 * PI);
    float v = 0.5 - asin(normalized.y) / PI;
    return vec2(u, v);
}
```

---

### 3. 粒子渲染最佳實踐

**Task**: 研究 Three.js Points 的效能最佳化

**Decision**: 使用 BufferGeometry + ShaderMaterial

**Rationale**:
- BufferGeometry 直接操作 GPU 緩衝區，效能最佳
- ShaderMaterial 允許自訂頂點和片段著色器
- 50,000 點在現代 GPU 上可輕鬆維持 60 FPS

**Best Practices**:
1. 預先分配 Float32Array，避免動態調整
2. 使用 `depthTest: false` 和 `depthWrite: false` 避免深度測試開銷
3. 使用 `AdditiveBlending` 產生發光效果
4. 設定合適的 `pointSize` 避免過度 overdraw

---

### 4. 攝影機平滑跟隨

**Task**: 實現平滑的滑鼠跟隨攝影機

**Decision**: 使用指數衰減插值

**Rationale**:
- 指數衰減 `pos += (target - pos) * factor` 產生自然的緩動
- 滑鼠離開時，target 回歸預設位置，自動產生返回動畫
- 因子 0.05 提供適當的跟隨速度

**Implementation**:
```javascript
// 在 animate() 中
const targetX = isMouseInWindow ? mouse.x * sensitivity : 0;
const targetY = isMouseInWindow ? mouse.y * sensitivity : 0;

camera.position.x += (targetX - camera.position.x) * 0.05;
camera.position.y += (targetY - camera.position.y) * 0.05;
camera.lookAt(center);
```

---

### 5. NASA Blue Marble 紋理來源

**Task**: 確認地球紋理圖的來源和授權

**Decision**: 使用 Three.js 範例庫中的地球紋理或 NASA 公開資源

**Rationale**:
- NASA 圖像為公共領域（Public Domain），無授權問題
- Three.js 範例庫已有現成的地球紋理可用
- 建議解析度：2048x1024 平衡品質與載入時間

**Source Options**:
1. Three.js examples: `https://threejs.org/examples/textures/planets/earth_atmos_2048.jpg`
2. NASA Visible Earth: https://visibleearth.nasa.gov/collection/1484/blue-marble

---

### 6. WebGL 可用性檢測

**Task**: 實現 WebGL 不支援時的降級方案

**Decision**: 使用 Three.js WebGL 檢測 + 靜態圖片降級

**Implementation**:
```javascript
import { WebGL } from 'three/addons/capabilities/WebGL.js';

if (WebGL.isWebGLAvailable()) {
    init();
} else {
    const fallback = document.createElement('div');
    fallback.innerHTML = `
        <img src="assets/earth-fallback.jpg" alt="Particle Earth" 
             style="max-width: 100%; height: auto;">
        <p>Your browser does not support WebGL.</p>
    `;
    document.body.appendChild(fallback);
}
```

---

## Summary

所有技術問題已解決：

| 問題 | 解決方案 |
|------|----------|
| 粒子分布 | 斐波那契球面分布（黃金角演算法） |
| UV 座標 | 等距柱狀投影公式 |
| 效能最佳化 | BufferGeometry + ShaderMaterial |
| 攝影機跟隨 | 指數衰減插值 |
| 紋理來源 | Three.js 範例庫 / NASA 公開資源 |
| WebGL 降級 | 靜態圖片替代 |

準備進入 Phase 1: 資料模型與契約設計。
