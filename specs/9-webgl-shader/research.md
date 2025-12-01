# Research: WebGL Shader 動態視覺效果

**Feature**: 9-webgl-shader  
**Date**: 2025-12-01  
**Status**: Complete

## 研究目標

解析 Three.js WebGL Shader 範例的技術實作，確保正確理解 GLSL shader 整合方式和最佳實踐。

---

## 研究項目

### 1. Three.js ShaderMaterial 使用方式

**問題**: 如何在 Three.js 中正確使用自訂 GLSL shader？

**發現**:
- 使用 `THREE.ShaderMaterial` 類別建立自訂著色器材質
- 需要提供 `vertexShader` 和 `fragmentShader` 兩個 GLSL 程式碼字串
- `uniforms` 物件用於傳遞動態參數給 shader

**決策**: 採用 ShaderMaterial 搭配內嵌 GLSL 程式碼

**理由**: 這是 Three.js 官方範例的標準做法，程式碼簡潔且易於理解

**替代方案考量**:
- 外部 .glsl 檔案：增加複雜度，需要額外的載入邏輯
- RawShaderMaterial：需要手動處理所有 Three.js 內建變數，過於複雜

---

### 2. 正交相機配置

**問題**: 為什麼使用 OrthographicCamera 而非 PerspectiveCamera？

**發現**:
- OrthographicCamera 沒有透視效果，適合 2D 渲染
- 設定 `left: -1, right: 1, top: 1, bottom: -1` 可建立標準化的 [-1, 1] 座標空間
- 搭配 `PlaneGeometry(2, 2)` 可精確填滿整個視窗

**決策**: 使用 `OrthographicCamera(-1, 1, 1, -1, 0, 1)`

**理由**: 這是全螢幕 shader 效果的標準配置，確保 UV 座標正確對應到螢幕空間

**替代方案考量**:
- PerspectiveCamera + 計算平面距離：過於複雜，無實際益處

---

### 3. Monjori Shader 分析

**問題**: Monjori shader 的運作原理是什麼？

**發現**:
- 作者: Mic (Pouët demoscene 社群)
- 類型: 程式化視覺藝術 (Procedural Art)
- 核心技術: 使用三角函數 (sin, cos, tan) 和數學運算產生動態圖案
- 輸入: UV 座標 (0-1) 和時間參數
- 輸出: RGB 顏色值

**Shader 結構分析**:

```glsl
// Vertex Shader
varying vec2 vUv;           // 傳遞 UV 到 fragment shader
void main() {
    vUv = uv;               // 內建 uv 屬性
    gl_Position = vec4(position, 1.0);  // 直接使用位置，無變換
}

// Fragment Shader
varying vec2 vUv;
uniform float time;         // 時間參數 (秒)
void main() {
    vec2 p = -1.0 + 2.0 * vUv;  // 將 UV 從 [0,1] 轉換到 [-1,1]
    float a = time * 40.0;      // 時間縮放因子
    // ... 複雜的數學運算產生動態圖案
    gl_FragColor = vec4(..., 1.0);  // 輸出顏色
}
```

**決策**: 直接使用原始 Monjori shader 程式碼

**理由**: 這是經典的 demoscene 作品，保持原始實作可確保視覺效果正確

---

### 4. 動畫迴圈實作

**問題**: 如何正確實作持續更新的動畫迴圈？

**發現**:
- Three.js r127+ 引入 `renderer.setAnimationLoop(callback)`
- 比傳統 `requestAnimationFrame` 更簡潔
- 自動處理 WebXR 相容性

**決策**: 使用 `renderer.setAnimationLoop(animate)`

**理由**: 這是 Three.js 目前推薦的動畫迴圈 API

**替代方案考量**:
- requestAnimationFrame: 仍可用但較冗長，需要手動遞迴呼叫

---

### 5. 時間參數來源

**問題**: 應該使用什麼作為時間來源？

**發現**:
- `performance.now()`: 高精度時間戳（毫秒），從頁面載入開始計算
- `Date.now()`: 低精度，不適合動畫
- `clock.getElapsedTime()`: Three.js Clock 類別，內部也使用 performance.now

**決策**: 使用 `performance.now() / 1000` 轉換為秒

**理由**: 直接簡潔，官方範例的標準做法

---

### 6. 視窗調整處理

**問題**: 視窗大小改變時需要更新哪些元件？

**發現**:
- 正交相機參數固定為 [-1, 1]，不需要根據視窗比例調整
- 只需更新 renderer 的尺寸

**決策**: 僅呼叫 `renderer.setSize(window.innerWidth, window.innerHeight)`

**理由**: 由於使用標準化座標空間，相機不需要調整

---

## 技術風險評估

| 風險 | 可能性 | 影響 | 緩解措施 |
|------|--------|------|----------|
| WebGL 不支援 | 低 | 高 | 範圍外，顯示空白頁面 |
| CDN 不可用 | 低 | 高 | 可考慮本地備份 |
| Shader 編譯錯誤 | 低 | 高 | 使用已驗證的原始程式碼 |

---

## 參考資源

- [Three.js ShaderMaterial 文件](https://threejs.org/docs/#api/en/materials/ShaderMaterial)
- [Three.js 官方 webgl_shader 範例](https://github.com/mrdoob/three.js/blob/master/examples/webgl_shader.html)
- [Monjori 原作 (Pouët)](http://www.pouet.net/prod.php?which=52761)
- [GLSL 程式設計指南](https://thebookofshaders.com/)
