# Research: WebGL Video Kinect 深度影片點雲視覺化

**Feature**: 10-video-kinect  
**Date**: 2025-12-01  
**Status**: Complete

## 研究目標

解析 Three.js WebGL Video Kinect 範例的技術實作，深入理解 Kinect 深度資料轉換為 3D 點雲的方法和最佳實踐。

---

## 研究項目

### 1. Kinect 深度影片格式

**問題**: Kinect 深度影片如何編碼深度資訊？

**發現**:
- Kinect 原始深度資料為 11-bit (0-2047) 或 16-bit
- 用於展示的深度影片將深度編碼為灰階值 (RGB 通道相同)
- 較亮的像素表示較近的物體，較暗表示較遠
- 影片解析度: 640x480 像素
- 支援格式: WebM (VP8/VP9) 和 MP4 (H.264)

**決策**: 使用 RGB 三通道平均值作為深度 `(r + g + b) / 3.0`

**理由**: 確保即使影片編碼有微小的色差，仍能獲得穩定的深度值

**替代方案考量**:
- 僅使用 R 通道：對編碼誤差較敏感
- 使用亮度公式 (0.299R + 0.587G + 0.114B)：不必要的複雜度

---

### 2. Kinect 視野參數 (FOV)

**問題**: 如何從 2D 像素座標轉換為 3D 空間座標？

**發現**:
- Kinect v1 的視野角度:
  - 水平 FOV: 約 57° (1.0144686 弧度)
  - 垂直 FOV: 約 43° (0.7898090 弧度)
- 轉換常數計算:
  ```
  XtoZ = tan(FOV_H / 2) * 2 = tan(1.0144686 / 2) * 2 ≈ 1.11146
  YtoZ = tan(FOV_V / 2) * 2 = tan(0.7898090 / 2) * 2 ≈ 0.83359
  ```

**決策**: 使用預計算的常數 XtoZ = 1.11146, YtoZ = 0.83359

**理由**: 避免 runtime 計算三角函數，提升效能

**轉換公式**:
```glsl
// 標準化像素座標到 [-0.5, 0.5]
float normalizedX = pixelX / width - 0.5;
float normalizedY = pixelY / height - 0.5;

// 使用深度和視野參數計算 3D 座標
float x = normalizedX * z * XtoZ;
float y = normalizedY * z * YtoZ;
float z = (1.0 - depth) * (far - near) + near;
```

---

### 3. 深度範圍與裁切

**問題**: Kinect 有效深度範圍是多少？如何處理無效深度？

**發現**:
- Kinect v1 有效感測範圍: 約 800mm - 4000mm
- 過近物體 (< 800mm): 深度值不可靠或缺失
- 過遠物體 (> 4000mm): 深度值飽和
- 深度值 0 通常表示無效測量

**決策**: 
- nearClipping 預設 850mm (略高於最小有效範圍)
- farClipping 預設 4000mm (最大有效範圍)
- 透過 GUI 允許用戶調整這些參數

**理由**: 提供合理預設值，同時保留彈性讓用戶根據實際影片內容調整

---

### 4. THREE.Points 與 BufferGeometry

**問題**: 如何高效渲染 307,200 個點？

**發現**:
- `THREE.Points` 是專為點雲渲染設計的類別
- `BufferGeometry` 直接操作 GPU buffer，效能最佳
- 頂點位置在 Vertex Shader 中動態計算，無需每幀更新 CPU 端資料

**BufferGeometry 初始化策略**:
```javascript
const vertices = new Float32Array(width * height * 3);
for (let i = 0, j = 0; i < vertices.length; i += 3, j++) {
    vertices[i] = j % width;      // 像素 X 座標
    vertices[i + 1] = Math.floor(j / width);  // 像素 Y 座標
    vertices[i + 2] = 0;          // Z 座標 (在 shader 中計算)
}
```

**決策**: 使用 BufferGeometry 儲存像素座標，3D 位置在 GPU 計算

**理由**: 
- 初始化一次，之後不需更新 CPU 端資料
- GPU 並行計算所有點的位置
- 記憶體效率高

---

### 5. VideoTexture 配置

**問題**: 如何正確設定 VideoTexture 供 Shader 使用？

**發現**:
- `THREE.VideoTexture` 自動從 HTML video 元素建立紋理
- 需要禁用 mipmap 產生以避免模糊
- 使用 NearestFilter 確保準確讀取像素值

**最佳配置**:
```javascript
const texture = new THREE.VideoTexture(video);
texture.minFilter = THREE.NearestFilter;
texture.magFilter = THREE.NearestFilter;  // 可選，但建議
texture.generateMipmaps = false;
```

**決策**: 使用 NearestFilter 和禁用 mipmaps

**理由**: 深度影片的每個像素都代表獨立的深度值，不應進行插值或混合

---

### 6. 加法混合模式 (Additive Blending)

**問題**: 為什麼使用加法混合而非標準混合？

**發現**:
- 標準混合 (NormalBlending): 後渲染的物體遮擋前面的
- 加法混合 (AdditiveBlending): 顏色值相加，產生發光效果
- 深度測試與深度寫入通常需禁用以獲得最佳效果

**視覺效果比較**:
| 混合模式 | 效果 | 適用場景 |
|----------|------|----------|
| Normal | 實體、不透明 | 標準 3D 物件 |
| Additive | 發光、半透明疊加 | 粒子、光效、點雲 |

**決策**: 使用 AdditiveBlending + depthTest: false + depthWrite: false

**理由**: 產生夢幻般的發光效果，符合原始範例的視覺風格

---

### 7. 相機跟隨滑鼠

**問題**: 如何實現平滑的相機跟隨效果？

**發現**:
- 直接設定相機位置會產生跳躍感
- 使用線性插值 (lerp) 可實現平滑過渡
- 插值因子越小，跟隨越慢、越平滑

**實作方式**:
```javascript
// 滑鼠位置轉換為目標座標
mouse.x = (clientX - windowWidth / 2) * 8;
mouse.y = (clientY - windowHeight / 2) * 8;

// 動畫迴圈中進行平滑插值
camera.position.x += (mouse.x - camera.position.x) * 0.05;
camera.position.y += (-mouse.y - camera.position.y) * 0.05;
camera.lookAt(center);
```

**決策**: 使用 0.05 的插值因子

**理由**: 在響應速度和平滑度之間取得平衡，符合原始範例的手感

---

### 8. lil-gui 整合

**問題**: 如何將 GUI 控制項與 Shader uniforms 連接？

**發現**:
- lil-gui 可直接綁定物件屬性
- ShaderMaterial 的 uniforms 結構為 `{ paramName: { value: x } }`
- GUI 可直接操作 `uniform.paramName.value`

**實作方式**:
```javascript
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';

const gui = new GUI();
gui.add(material.uniforms.nearClipping, 'value', 1, 10000, 1.0)
   .name('nearClipping');
gui.add(material.uniforms.farClipping, 'value', 1, 10000, 1.0)
   .name('farClipping');
gui.add(material.uniforms.pointSize, 'value', 1, 10, 1.0)
   .name('pointSize');
gui.add(material.uniforms.zOffset, 'value', 0, 4000, 1.0)
   .name('zOffset');
```

**決策**: 直接綁定 uniforms.xxx.value 屬性

**理由**: 最簡潔的方式，變更立即反映在下一幀渲染

---

## 總結

### 關鍵技術決策一覽

| 項目 | 決策 | 理由 |
|------|------|------|
| 深度解析 | RGB 平均值 | 穩定、簡單 |
| 視野常數 | 預計算 XtoZ/YtoZ | 效能優化 |
| 頂點資料 | 像素座標存 GPU | 減少 CPU-GPU 傳輸 |
| 紋理過濾 | NearestFilter | 保持深度精確度 |
| 混合模式 | Additive | 視覺效果需求 |
| 相機跟隨 | 0.05 插值因子 | 平滑且響應 |
| GUI 綁定 | 直接操作 uniform.value | 簡潔有效 |

### 效能考量

- **GPU 運算**: 307,200 個頂點在 GPU 並行計算
- **CPU 負擔**: 僅滑鼠追蹤和相機更新
- **記憶體**: 約 3.7MB 頂點資料 (307,200 * 3 * 4 bytes)
- **預期 FPS**: 現代顯卡應可達 60fps，最低保證 30fps
