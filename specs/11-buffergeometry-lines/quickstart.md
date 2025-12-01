# Quickstart: WebGL BufferGeometry Lines

**Feature**: 11-buffergeometry-lines  
**Date**: 2025-12-01

---

## 概述

此範例展示如何使用 Three.js 的 BufferGeometry 建立大量線段，並透過頂點顏色和 morph targets 實現動態視覺效果。

---

## 快速開始

### 1. 檔案結構

```
examples/
└── webgl-buffergeometry-lines/
    ├── index.html      # 主要展示頁面
    └── fallback.png    # WebGL 不支援時的替代圖片
```

### 2. 執行方式

由於使用 ES Modules，需要透過 HTTP 伺服器執行：

```bash
# 使用 Python 3
cd examples/webgl-buffergeometry-lines
python3 -m http.server 8080

# 或使用 Node.js (npx)
npx serve .

# 然後開啟瀏覽器
open http://localhost:8080
```

### 3. 預期結果

- 畫面中央顯示由 10,000 條彩色線段組成的 3D 物件
- 物件持續旋轉
- 線段形態週期性變形
- 左上角顯示 FPS 統計

---

## 核心概念

### BufferGeometry 頂點資料

```javascript
// 建立頂點位置和顏色陣列
const positions = [];
const colors = [];

for (let i = 0; i < 10000; i++) {
    // 隨機位置
    const x = Math.random() * 800 - 400;
    const y = Math.random() * 800 - 400;
    const z = Math.random() * 800 - 400;
    positions.push(x, y, z);
    
    // 根據位置計算顏色
    colors.push((x / 800) + 0.5);  // R
    colors.push((y / 800) + 0.5);  // G
    colors.push((z / 800) + 0.5);  // B
}

// 設定幾何屬性
geometry.setAttribute('position', 
    new THREE.Float32BufferAttribute(positions, 3));
geometry.setAttribute('color', 
    new THREE.Float32BufferAttribute(colors, 3));
```

### Morph Targets

```javascript
// 產生另一組隨機位置作為目標形態
const morphData = [];
for (let i = 0; i < 10000; i++) {
    morphData.push(
        Math.random() * 800 - 400,
        Math.random() * 800 - 400,
        Math.random() * 800 - 400
    );
}

const morphTarget = new THREE.Float32BufferAttribute(morphData, 3);
geometry.morphAttributes.position = [morphTarget];

// 動畫中更新權重 (0-1)
line.morphTargetInfluences[0] = Math.abs(Math.sin(time));
```

### Timer 計時器

```javascript
// 建立計時器並連接 document（頁面不可見時暫停）
const timer = new THREE.Timer();
timer.connect(document);

function animate() {
    timer.update();
    
    const delta = timer.getDelta();    // 每幀時間差
    const time = timer.getElapsed();   // 總經過時間
    
    // 使用時間控制動畫
    line.rotation.y = time * 0.5;
}
```

---

## 自訂選項

### 調整線段數量

修改 `segments` 常數（注意效能影響）：

```javascript
const segments = 10000;  // 預設值
const segments = 5000;   // 較低效能需求
const segments = 20000;  // 較高效能需求
```

### 調整分布範圍

修改 `r` 常數：

```javascript
const r = 800;   // 預設值
const r = 1200;  // 更分散
const r = 400;   // 更集中
```

### 調整旋轉速度

修改 animate() 中的係數：

```javascript
line.rotation.x = time * 0.25;  // X 軸旋轉速度
line.rotation.y = time * 0.5;   // Y 軸旋轉速度
```

### 調整 morph 速度

修改時間累加係數：

```javascript
t += delta * 0.5;  // morph 動畫速度
```

---

## 疑難排解

### 問題：畫面空白

**可能原因**：
1. 瀏覽器不支援 WebGL
2. 未透過 HTTP 伺服器執行

**解決方案**：
1. 檢查是否顯示 fallback 圖片和錯誤訊息
2. 確認使用現代瀏覽器
3. 確認透過 localhost 或其他 HTTP 伺服器存取

### 問題：效能不佳

**可能原因**：
1. 硬體不支援 GPU 加速
2. 線段數量過多

**解決方案**：
1. 減少 `segments` 數量
2. 關閉瀏覽器其他分頁
3. 確認驅動程式已更新

---

## 相關資源

- [Three.js 官方範例](https://threejs.org/examples/#webgl_buffergeometry_lines)
- [BufferGeometry 文件](https://threejs.org/docs/#api/en/core/BufferGeometry)
- [Morph Targets 文件](https://threejs.org/docs/#api/en/core/BufferGeometry.morphAttributes)
