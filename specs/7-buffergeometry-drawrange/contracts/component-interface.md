# Component Interface: BufferGeometry DrawRange

**Date**: 2025-11-28  
**Source**: [data-model.md](../data-model.md)

---

## 模組結構

本功能為單一 HTML 檔案，包含內嵌的 JavaScript 模組。以下定義主要函式介面。

---

## 全域變數

```javascript
// Three.js 核心物件
let container, stats;
let camera, scene, renderer;
let group;

// 粒子系統
let particles;           // THREE.BufferGeometry
let pointCloud;          // THREE.Points
let particlePositions;   // Float32Array

// 連線系統
let positions;           // Float32Array (線段頂點)
let colors;              // Float32Array (線段顏色)
let linesMesh;           // THREE.LineSegments

// 資料陣列
const particlesData = []; // Array<ParticleData>

// 常數
const maxParticleCount = 1000;
let particleCount = 500;
const r = 800;
const rHalf = r / 2;
```

---

## 介面定義

### effectController

GUI 控制器物件。

```typescript
interface EffectController {
  showDots: boolean;        // 顯示粒子 (預設: true)
  showLines: boolean;       // 顯示連線 (預設: true)
  minDistance: number;      // 連線距離門檻 (預設: 150, 範圍: 10-300)
  limitConnections: boolean; // 限制連接數 (預設: false)
  maxConnections: number;   // 最大連接數 (預設: 20, 範圍: 0-30)
  particleCount: number;    // 粒子數量 (預設: 500, 範圍: 0-1000)
}
```

### ParticleData

單一粒子的運動資料。

```typescript
interface ParticleData {
  velocity: THREE.Vector3;  // 速度向量 (各分量: -1 到 1)
  numConnections: number;   // 當前連接數量 (每幀重置)
}
```

---

## 函式簽章

### initGUI()

初始化 GUI 控制面板。

```typescript
function initGUI(): void
```

**行為**:
- 建立 lil-gui 實例
- 綁定 effectController 的所有屬性
- 設定 onChange 回呼處理即時更新

**副作用**:
- 將 GUI 附加到 DOM
- 修改 pointCloud.visible、linesMesh.visible
- 修改 particleCount 和 particles.drawRange

---

### init()

初始化整個場景。

```typescript
function init(): void
```

**行為**:
1. 呼叫 initGUI()
2. 取得 container DOM 元素
3. 建立 PerspectiveCamera
4. 建立 OrbitControls
5. 建立 Scene 和 Group
6. 建立 BoxHelper（邊界框）
7. 初始化粒子位置陣列和 particlesData
8. 建立粒子 BufferGeometry 和 Points
9. 建立連線 BufferGeometry 和 LineSegments
10. 建立 WebGLRenderer
11. 建立 Stats
12. 綁定 resize 事件

**副作用**:
- 修改所有全域變數
- 操作 DOM

---

### onWindowResize()

處理視窗大小變化。

```typescript
function onWindowResize(): void
```

**行為**:
- 更新 camera.aspect
- 呼叫 camera.updateProjectionMatrix()
- 呼叫 renderer.setSize()

---

### animate()

動畫迴圈（每幀呼叫）。

```typescript
function animate(): void
```

**行為**:
1. 重置所有 particlesData[i].numConnections
2. 更新粒子位置（加上速度）
3. 邊界檢查與速度反轉
4. 檢查連接限制條件
5. 計算粒子間距離
6. 建立連線頂點和顏色資料
7. 更新 geometry drawRange 和 needsUpdate
8. 呼叫 render()
9. 更新 Stats

---

### render()

渲染場景。

```typescript
function render(): void
```

**行為**:
- 計算旋轉角度（基於時間）
- 設定 group.rotation.y
- 呼叫 renderer.render(scene, camera)

---

## 事件處理

### resize

```typescript
window.addEventListener('resize', onWindowResize);
```

### GUI onChange 回呼

| 控制項 | 回呼行為 |
|--------|---------|
| showDots | `pointCloud.visible = value` |
| showLines | `linesMesh.visible = value` |
| minDistance | 無直接回呼，在 animate() 中讀取 |
| limitConnections | 無直接回呼，在 animate() 中讀取 |
| maxConnections | 無直接回呼，在 animate() 中讀取 |
| particleCount | `particleCount = value; particles.setDrawRange(0, particleCount)` |

---

## 錯誤處理

### WebGL 支援檢查

```javascript
if (!renderer) {
  // 顯示錯誤訊息
  container.innerHTML = '<p>您的瀏覽器不支援 WebGL</p>';
  return;
}
```

### 邊界安全

```javascript
// 確保粒子不會超出邊界
if (particlePositions[i * 3] < -rHalf || particlePositions[i * 3] > rHalf)
  particleData.velocity.x = -particleData.velocity.x;
```

---

## 效能考量

1. **避免每幀建立物件**: 使用預分配的 Float32Array
2. **批次更新**: 所有位置和顏色更新後，才設定 needsUpdate
3. **drawRange 優化**: 只渲染實際使用的頂點
4. **提前跳出**: 當達到 maxConnections 時跳過後續計算
