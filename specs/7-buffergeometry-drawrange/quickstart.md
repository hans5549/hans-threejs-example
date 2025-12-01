# Quickstart: BufferGeometry DrawRange 互動式粒子網絡

**Date**: 2025-11-28

---

## 概述

這是一個 Three.js 互動式 3D 粒子網絡視覺化展示，展示 BufferGeometry 和 drawRange 的動態渲染技術。

## 快速啟動

### 1. 開啟範例

```bash
# 使用任何靜態伺服器
cd examples/buffergeometry-drawrange
python -m http.server 8000
# 或
npx serve .
```

然後在瀏覽器開啟: `http://localhost:8000`

### 2. 直接開啟

由於使用 CDN 載入相依套件，也可以直接在瀏覽器開啟 `index.html` 檔案。

---

## 功能說明

### 粒子動畫

- 場景中有最多 1000 個白色粒子
- 粒子在立方體邊界內自由移動
- 碰到邊界會自動反彈
- 場景會緩慢自動旋轉

### 動態連線

- 距離接近的粒子會自動產生連線
- 連線透明度隨距離變化（越近越不透明）
- 使用 Additive Blending 產生發光效果

### GUI 控制面板

右上角的控制面板可調整：

| 控制項 | 說明 |
|--------|------|
| showDots | 顯示/隱藏粒子 |
| showLines | 顯示/隱藏連線 |
| minDistance | 連線產生的距離門檻 (10-300) |
| limitConnections | 是否限制每個粒子的連接數 |
| maxConnections | 每個粒子的最大連接數 (0-30) |
| particleCount | 顯示的粒子數量 (0-1000) |

### 攝影機控制

- **左鍵拖曳**: 旋轉視角
- **滾輪**: 縮放 (距離限制: 1000-3000)
- **右鍵拖曳**: 平移視角

---

## 技術重點

### BufferGeometry 動態更新

```javascript
// 設定動態使用模式
geometry.setAttribute('position', 
  new THREE.BufferAttribute(positions, 3).setUsage(THREE.DynamicDrawUsage)
);

// 每幀更新後通知 GPU
geometry.attributes.position.needsUpdate = true;

// 控制實際渲染數量
geometry.setDrawRange(0, actualCount);
```

### 距離計算與連線生成

```javascript
for (let i = 0; i < particleCount; i++) {
  for (let j = i + 1; j < particleCount; j++) {
    const dx = positions[i * 3] - positions[j * 3];
    const dy = positions[i * 3 + 1] - positions[j * 3 + 1];
    const dz = positions[i * 3 + 2] - positions[j * 3 + 2];
    const dist = Math.sqrt(dx * dx + dy * dy + dz * dz);
    
    if (dist < minDistance) {
      const alpha = 1.0 - dist / minDistance;
      // 建立連線...
    }
  }
}
```

---

## 相依套件

- Three.js r160
- lil-gui（控制面板）
- Stats.js（效能監測）
- OrbitControls（攝影機控制）

所有套件透過 CDN 載入，無需本地安裝。

---

## 瀏覽器支援

需要支援 WebGL 的現代瀏覽器：
- Chrome 60+
- Firefox 55+
- Safari 12+
- Edge 79+

---

## 參考資料

- [Three.js 官方範例](https://threejs.org/examples/#webgl_buffergeometry_drawrange)
- [BufferGeometry 文件](https://threejs.org/docs/#api/en/core/BufferGeometry)
- [drawRange 說明](https://threejs.org/docs/#api/en/core/BufferGeometry.drawRange)
