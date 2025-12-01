# Research: WebGL BufferGeometry Lines

**Feature**: 11-buffergeometry-lines  
**Date**: 2025-12-01  
**Status**: Complete

---

## 1. BufferGeometry 頂點屬性設定

### Decision
使用 `THREE.Float32BufferAttribute` 建立 position 和 color 屬性，每個屬性的 itemSize 設為 3（對應 x/y/z 或 r/g/b）。

### Rationale
- Float32BufferAttribute 是 Three.js 標準的頂點資料容器
- itemSize=3 符合 3D 座標 (x, y, z) 和 RGB 顏色 (r, g, b) 的資料結構
- 使用 vertexColors 材質屬性時，每個頂點需要對應的顏色資料

### Alternatives Considered
- **Float64BufferAttribute**: 精度過高，效能較差，非必要
- **InterleavedBufferAttribute**: 適合更複雜的資料結構，本例不需要

### Implementation Pattern
```javascript
const positions = [];
const colors = [];

for (let i = 0; i < segments; i++) {
    positions.push(x, y, z);
    colors.push(r, g, b);
}

geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3));
geometry.computeBoundingSphere();
```

---

## 2. Morph Targets 實作機制

### Decision
使用 `geometry.morphAttributes.position` 陣列儲存目標形態，透過 `line.morphTargetInfluences[0]` 控制變形權重。

### Rationale
- Three.js 的 morph targets 機制允許平滑的頂點位置插值
- 權重值 0-1 之間，GPU 自動計算中間狀態
- 使用正弦函數產生週期性的權重變化

### Implementation Pattern
```javascript
function generateMorphTargets(geometry) {
    const data = [];
    for (let i = 0; i < segments; i++) {
        data.push(
            Math.random() * r - r / 2,
            Math.random() * r - r / 2,
            Math.random() * r - r / 2
        );
    }
    const morphTarget = new THREE.Float32BufferAttribute(data, 3);
    morphTarget.name = 'target1';
    geometry.morphAttributes.position = [morphTarget];
}

// 動畫中更新權重
line.morphTargetInfluences[0] = Math.abs(Math.sin(t));
```

---

## 3. THREE.Timer 計時器 API

### Decision
使用 `THREE.Timer` 類別管理動畫時間，搭配 `connect(document)` 實現頁面可見性感知。

### Rationale
- `timer.connect(document)` 會在頁面不可見時自動暫停計時，節省資源
- `getDelta()` 回傳上一幀到這一幀的時間差（秒）
- `getElapsed()` 回傳從開始到現在的總經過時間（秒）
- 分離 delta 和 elapsed 允許獨立控制旋轉和 morph 動畫

### Implementation Pattern
```javascript
const timer = new THREE.Timer();
timer.connect(document);

function animate() {
    timer.update();
    const delta = timer.getDelta();
    const time = timer.getElapsed();
    
    line.rotation.x = time * 0.25;
    line.rotation.y = time * 0.5;
    
    t += delta * 0.5;
    line.morphTargetInfluences[0] = Math.abs(Math.sin(t));
}
```

---

## 4. WebGL 支援偵測

### Decision
使用 try-catch 包裹 WebGLRenderer 建立，失敗時顯示 fallback 圖片和錯誤訊息。

### Rationale
- 現代瀏覽器大多支援 WebGL，但仍需處理例外情況
- 提供替代內容確保使用者能理解頁面用途
- 符合漸進增強的網頁設計原則

### Alternatives Considered
- **THREE.WEBGL.isWebGLAvailable()**: Three.js 提供的檢測函式，但可能無法捕捉所有失敗情況
- **僅顯示錯誤文字**: 使用者體驗較差

### Implementation Pattern
```javascript
function init() {
    try {
        renderer = new THREE.WebGLRenderer();
        // ... 正常初始化
    } catch (e) {
        showFallback();
        return;
    }
}

function showFallback() {
    const container = document.getElementById('container');
    container.innerHTML = `
        <div class="fallback">
            <img src="fallback.png" alt="WebGL BufferGeometry Lines 展示">
            <p>您的瀏覽器不支援 WebGL。請使用現代瀏覽器（Chrome, Firefox, Safari, Edge）以獲得最佳體驗。</p>
        </div>
    `;
}
```

---

## 5. 顏色計算公式

### Decision
使用位置座標正規化後的值作為 RGB 分量：`color = (position / range) + 0.5`

### Rationale
- 位置範圍 [-r/2, r/2] 映射到顏色範圍 [0, 1]
- 產生從角落到中心的漸層效果
- 簡單的線性映射，計算效率高

### Color Distribution
| 位置區域 | 近似顏色 |
|---------|---------|
| (-r/2, -r/2, -r/2) | 黑色 (0, 0, 0) |
| (+r/2, +r/2, +r/2) | 白色 (1, 1, 1) |
| (0, 0, 0) | 灰色 (0.5, 0.5, 0.5) |
| (+r/2, -r/2, 0) | 品紅 (1, 0, 0.5) |

---

## 6. 相機設定參數

### Decision
使用透視相機，FOV=27°，位置 Z=2750，近平面=1，遠平面=4000。

### Rationale
- 窄視角 (27°) 減少透視變形，適合觀看密集的線段
- Z=2750 的距離確保整個 r=800 的立方體在視野內
- 近/遠平面設定涵蓋所有可能的物件位置

---

## Summary

所有研究項目已完成，無待澄清事項。技術方案清晰，可進入 Phase 1 設計階段。
