# Research: WebGL Custom Attributes Lines

**Feature**: 12-custom-attributes-lines  
**Date**: 2025-12-01  
**Status**: Complete

## 1. ShaderMaterial 與自訂 Attributes 機制

### 決策
使用 `THREE.ShaderMaterial` 配合 `BufferAttribute` 實現自訂頂點屬性。

### 理由
- ShaderMaterial 提供完全的著色器控制能力
- BufferAttribute 直接對應 GPU 緩衝區，效能最佳
- Three.js 自動處理 attribute 變數綁定

### 實作細節

**Vertex Shader 中的 Attribute 宣告：**
```glsl
attribute vec3 displacement;
attribute vec3 customColor;
```

**JavaScript 中的 BufferAttribute 設定：**
```javascript
const count = geometry.attributes.position.count;
const displacement = new THREE.Float32BufferAttribute(count * 3, 3);
const customColor = new THREE.Float32BufferAttribute(count * 3, 3);
geometry.setAttribute('displacement', displacement);
geometry.setAttribute('customColor', customColor);
```

**Uniforms 定義：**
```javascript
uniforms = {
  amplitude: { value: 5.0 },
  opacity: { value: 0.3 },
  color: { value: new THREE.Color(0xffffff) }
};
```

### 替代方案考量
- PointsMaterial / LineBasicMaterial：功能受限，無法實現自訂位移效果
- RawShaderMaterial：過於底層，需自行處理更多內建變數

---

## 2. TextGeometry 與 FontLoader

### 決策
使用 `FontLoader` 載入 Helvetiker Bold 字型，建立 `TextGeometry`。

### 理由
- TextGeometry 提供完整的 3D 文字建模功能
- Helvetiker 是 Three.js 內建支援的標準字型
- CDN 提供穩定的字型檔案來源

### 實作細節

**字型載入：**
```javascript
const loader = new FontLoader();
loader.load('fonts/helvetiker_bold.typeface.json', function(font) {
  init(font);
}, undefined, function(error) {
  // 使用備用幾何體
  initWithFallback();
});
```

**TextGeometry 參數：**
```javascript
const geometry = new TextGeometry('three.js', {
  font: font,
  size: 50,
  depth: 15,
  curveSegments: 10,
  bevelThickness: 5,
  bevelSize: 1.5,
  bevelEnabled: true,
  bevelSegments: 10
});
geometry.center();
```

### 字型來源
- CDN: `https://cdn.jsdelivr.net/npm/three@0.174.0/examples/fonts/helvetiker_bold.typeface.json`

### 替代方案考量
- SVGLoader + ExtrudeGeometry：需要額外的 SVG 資源
- 內建 TextTexture：僅 2D，無法建立 3D 線條

---

## 3. Additive Blending 渲染

### 決策
使用 `THREE.AdditiveBlending` 配合 `depthTest: false` 和 `transparent: true`。

### 理由
- Additive Blending 創造發光疊加效果，適合線條視覺化
- 關閉深度測試避免線條遮擋問題
- 透明模式允許顏色混合

### 實作細節

**ShaderMaterial 配置：**
```javascript
const shaderMaterial = new THREE.ShaderMaterial({
  uniforms: uniforms,
  vertexShader: vertexShaderCode,
  fragmentShader: fragmentShaderCode,
  blending: THREE.AdditiveBlending,
  depthTest: false,
  transparent: true
});
```

### 視覺效果
- 線條重疊處顏色加亮
- 整體呈現柔和發光感
- opacity uniform 控制整體透明度

### 替代方案考量
- NormalBlending：無發光效果
- SubtractiveBlending：顏色變暗，不適合本展示

---

## 4. 動畫迴圈與效能

### 決策
使用 `renderer.setAnimationLoop()` 搭配 `Date.now()` 計時。

### 理由
- setAnimationLoop 自動處理 requestAnimationFrame
- Date.now() 提供毫秒級時間基準
- 與官方範例保持一致

### 實作細節

**動畫更新邏輯：**
```javascript
function render() {
  const time = Date.now() * 0.001;
  
  // 更新旋轉
  line.rotation.y = 0.25 * time;
  
  // 更新 amplitude uniform (正弦波動)
  uniforms.amplitude.value = Math.sin(0.5 * time);
  
  // 更新顏色 (HSL 漸變)
  uniforms.color.value.offsetHSL(0.0005, 0, 0);
  
  // 更新 displacement attribute
  const array = line.geometry.attributes.displacement.array;
  for (let i = 0; i < array.length; i += 3) {
    array[i] += 0.3 * (0.5 - Math.random());
    array[i + 1] += 0.3 * (0.5 - Math.random());
    array[i + 2] += 0.3 * (0.5 - Math.random());
  }
  line.geometry.attributes.displacement.needsUpdate = true;
  
  renderer.render(scene, camera);
}
```

### 效能考量
- displacement 陣列每幀更新需設定 `needsUpdate = true`
- 隨機數生成在合理範圍內不影響效能
- 頂點數量由 TextGeometry 決定，通常在數千級別

### 替代方案考量
- THREE.Clock：功能相似，但官方範例使用 Date.now()

---

## 5. 字型載入失敗處理

### 決策
FontLoader 失敗時使用 `BoxGeometry` 作為備用幾何體。

### 理由
- 確保使用者在字型載入失敗時仍可看到動畫效果
- BoxGeometry 簡單且頂點數量適中
- 保持展示的連續性

### 實作細節

**錯誤處理流程：**
```javascript
const loader = new FontLoader();
loader.load(
  fontUrl,
  function(font) {
    // 成功：使用 TextGeometry
    initWithText(font);
  },
  undefined,
  function(error) {
    console.warn('Font loading failed, using fallback geometry');
    // 失敗：使用 BoxGeometry
    initWithFallback();
  }
);

function initWithFallback() {
  const geometry = new THREE.BoxGeometry(100, 100, 100, 10, 10, 10);
  // 套用相同的 ShaderMaterial 和 attributes
  setupGeometryAttributes(geometry);
  line = new THREE.Line(geometry, shaderMaterial);
  scene.add(line);
}
```

### 替代方案考量
- 顯示錯誤訊息並停止：使用者體驗較差
- SphereGeometry：頂點分布不均勻
- TorusGeometry：形狀較複雜，線條效果較好（可作為進階選項）

---

## 研究結論

所有技術決策均已確認，可進入 Phase 1 設計階段。主要技術棧：

| 項目 | 選用技術 |
|------|----------|
| 材質 | THREE.ShaderMaterial |
| 幾何體 | TextGeometry (primary) / BoxGeometry (fallback) |
| 自訂屬性 | BufferAttribute (displacement, customColor) |
| 混合模式 | AdditiveBlending |
| 動畫 | setAnimationLoop + Date.now() |
| 字型 | Helvetiker Bold via CDN |
