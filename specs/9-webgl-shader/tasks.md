# Tasks: WebGL Shader 動態視覺效果

**Feature**: 9-webgl-shader  
**Date**: 2025-12-01  
**Plan**: [plan.md](./plan.md)

---

## 任務總覽

| ID | 任務 | 優先級 | 狀態 | 預估時間 |
|----|------|--------|------|----------|
| T1 | 建立基礎 HTML 結構 | P1 | ⬜ TODO | 15 min |
| T2 | 實作 GLSL Vertex Shader | P1 | ⬜ TODO | 10 min |
| T3 | 實作 GLSL Fragment Shader (Monjori) | P1 | ⬜ TODO | 15 min |
| T4 | 實作 Three.js 初始化 | P1 | ⬜ TODO | 20 min |
| T5 | 實作動畫迴圈 | P1 | ⬜ TODO | 10 min |
| T6 | 實作視窗調整處理 | P2 | ⬜ TODO | 10 min |
| T7 | 加入 UI 資訊區塊 | P3 | ⬜ TODO | 10 min |
| T8 | 視覺驗證測試 | - | ⬜ TODO | 15 min |

**總預估時間**: 約 1.5 小時

---

## 詳細任務

### T1: 建立基礎 HTML 結構

**優先級**: P1  
**相依性**: 無  
**產出**: `examples/webgl-shader/index.html`

**驗收標準**:
- [ ] HTML5 doctype
- [ ] 正確的 meta 標籤 (charset, viewport)
- [ ] 標題設定為 "three.js webgl - shader [Monjori]"
- [ ] 內嵌 CSS 樣式 (body margin, overflow)
- [ ] container div 元素
- [ ] Import map 配置 (Three.js CDN)

---

### T2: 實作 GLSL Vertex Shader

**優先級**: P1  
**相依性**: T1  
**產出**: `<script id="vertexShader">` 區塊

**實作內容**:
```glsl
varying vec2 vUv;

void main() {
    vUv = uv;
    gl_Position = vec4(position, 1.0);
}
```

**驗收標準**:
- [ ] 正確宣告 varying vUv
- [ ] 傳遞 uv 座標
- [ ] 輸出頂點位置

---

### T3: 實作 GLSL Fragment Shader (Monjori)

**優先級**: P1  
**相依性**: T1  
**產出**: `<script id="fragmentShader">` 區塊

**實作內容**:
- 宣告 `varying vec2 vUv`
- 宣告 `uniform float time`
- 實作 Monjori 演算法計算顏色

**驗收標準**:
- [ ] 正確接收 vUv varying
- [ ] 正確接收 time uniform
- [ ] 實作完整的 Monjori 顏色計算
- [ ] 輸出 gl_FragColor

---

### T4: 實作 Three.js 初始化

**優先級**: P1  
**相依性**: T1, T2, T3  
**產出**: `init()` 函式

**實作內容**:
1. 取得 container 元素
2. 建立 OrthographicCamera(-1, 1, 1, -1, 0, 1)
3. 建立 Scene
4. 建立 PlaneGeometry(2, 2)
5. 建立 uniforms 物件 `{ time: { value: 1.0 } }`
6. 建立 ShaderMaterial
7. 建立 Mesh 並加入 Scene
8. 建立 WebGLRenderer
9. 設定 pixelRatio 和 size
10. 將 canvas 加入 container
11. 設定 animationLoop
12. 註冊 resize 事件監聽器

**驗收標準**:
- [ ] 所有 Three.js 物件正確建立
- [ ] Canvas 正確插入 DOM
- [ ] 無控制台錯誤

---

### T5: 實作動畫迴圈

**優先級**: P1  
**相依性**: T4  
**產出**: `animate()` 函式

**實作內容**:
```javascript
function animate() {
    uniforms['time'].value = performance.now() / 1000;
    renderer.render(scene, camera);
}
```

**驗收標準**:
- [ ] time uniform 每幀更新
- [ ] 場景正確渲染
- [ ] 動畫持續運行不中斷

---

### T6: 實作視窗調整處理

**優先級**: P2  
**相依性**: T4  
**產出**: `onWindowResize()` 函式

**實作內容**:
```javascript
function onWindowResize() {
    renderer.setSize(window.innerWidth, window.innerHeight);
}
```

**驗收標準**:
- [ ] 視窗調整時自動更新 canvas 尺寸
- [ ] 效果在各種視窗比例下正確顯示

---

### T7: 加入 UI 資訊區塊

**優先級**: P3  
**相依性**: T1  
**產出**: `#info` div 元素

**實作內容**:
- Three.js 官網連結
- "shader demo" 文字
- Monjori 原作連結 (Pouët)

**驗收標準**:
- [ ] 資訊顯示在頁面頂部中央
- [ ] 連結在新分頁開啟
- [ ] 樣式清晰可讀

---

### T8: 視覺驗證測試

**優先級**: -  
**相依性**: T1-T7  
**產出**: 驗證報告

**測試項目**:
- [ ] 頁面載入後立即顯示動態效果
- [ ] 效果持續動態變化
- [ ] 色彩漸變流暢
- [ ] 調整視窗後效果正常
- [ ] 連結可點擊並開啟正確網站
- [ ] 無控制台錯誤或警告
- [ ] 在 Chrome 測試通過
- [ ] 在 Firefox 測試通過
- [ ] 在 Safari 測試通過 (如可用)

---

## 實作順序建議

```
T1 (HTML 結構)
├── T2 (Vertex Shader) ─┐
├── T3 (Fragment Shader)├── T4 (初始化)
└── T7 (UI 資訊)        │     ├── T5 (動畫)
                        │     └── T6 (Resize)
                        └── T8 (驗證)
```

**建議流程**:
1. 先完成 T1 建立基礎結構
2. 平行完成 T2, T3, T7
3. 完成 T4 整合所有元件
4. 完成 T5, T6 功能
5. 最後執行 T8 驗證
