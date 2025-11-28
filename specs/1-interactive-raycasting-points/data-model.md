# Data Model: Three.js 互動式點雲 Raycasting 範例

**Feature**: `1-interactive-raycasting-points`  
**Date**: 2025-11-28  
**Status**: Complete

## 實體關係圖

```
┌─────────────────────────────────────────────────────────────────┐
│                          Scene                                   │
│  (THREE.Scene - 包含所有 3D 物件的容器)                          │
└─────────────────────────────────────────────────────────────────┘
                                │
         ┌──────────────────────┼──────────────────────┐
         │                      │                      │
         ▼                      ▼                      ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  PointCloud[0]  │  │  PointCloud[1]  │  │  PointCloud[2]  │
│  (紅色-標準)     │  │  (綠色-索引)     │  │  (青色-分組)     │
│  position:      │  │  position:      │  │  position:      │
│    (-5, 0, 0)   │  │    (0, 0, 0)    │  │    (5, 0, 0)    │
│  scale:         │  │  scale:         │  │  scale:         │
│    (5, 10, 10)  │  │    (5, 10, 10)  │  │    (5, 10, 10)  │
└─────────────────┘  └─────────────────┘  └─────────────────┘
         │                      │                      │
         └──────────────────────┼──────────────────────┘
                                │
                                ▼
                    ┌─────────────────────┐
                    │  Sphere Markers     │
                    │  (物件池: 40 個)     │
                    │  - position: 動態   │
                    │  - scale: 動態縮放  │
                    └─────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        Raycaster                                 │
│  - origin: 攝影機位置                                            │
│  - direction: 滑鼠指向的 3D 方向                                 │
│  - threshold: 0.1                                                │
│                                                                  │
│  ◄─── pointer (滑鼠 2D 座標轉換)                                 │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
                    ┌─────────────────────┐
                    │    Intersection     │
                    │  - point: Vector3   │
                    │  - index: number    │
                    │  - object: Points   │
                    └─────────────────────┘
```

---

## 核心實體定義

### 1. Scene (場景)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `children` | Object3D[] | 場景中的所有物件 |
| `background` | Color \| null | 背景顏色 (預設: 黑色) |

### 2. Camera (攝影機)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `fov` | number | 視野角度: 45° |
| `aspect` | number | 長寬比: window.innerWidth / window.innerHeight |
| `near` | number | 近裁剪面: 1 |
| `far` | number | 遠裁剪面: 10000 |
| `position` | Vector3 | 初始位置: (10, 10, 10) |

### 3. PointCloud (點雲)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `geometry` | BufferGeometry | 點雲幾何資料 |
| `material` | PointsMaterial | 點雲材質 |
| `position` | Vector3 | 世界座標位置 |
| `scale` | Vector3 | 縮放比例: (5, 10, 10) |

#### BufferGeometry Attributes

| Attribute | 類型 | 元素大小 | 說明 |
|-----------|------|----------|------|
| `position` | Float32Array | 3 | 每個點的 (x, y, z) 座標 |
| `color` | Float32Array | 3 | 每個點的 (r, g, b) 顏色 |
| `index` | Uint16Array | 1 | 索引陣列 (僅 Indexed 類型) |

### 4. PointsMaterial (點材質)

| 屬性 | 類型 | 值 | 說明 |
|------|------|-----|------|
| `size` | number | 0.05 | 點的大小 |
| `vertexColors` | boolean | true | 啟用頂點顏色 |

### 5. Sphere Marker (球體標記)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `geometry` | SphereGeometry | 半徑: 0.1, 分段: 32×32 |
| `material` | MeshBasicMaterial | 顏色: 0xff0000 (紅色) |
| `position` | Vector3 | 動態更新為交集點位置 |
| `scale` | Vector3 | 動態縮放 (1.0 → 0.01) |

### 6. Raycaster (射線投射器)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `params.Points.threshold` | number | 0.1 - 偵測靈敏度 |

### 7. Pointer (滑鼠座標)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `x` | number | 正規化座標 [-1, 1] |
| `y` | number | 正規化座標 [-1, 1] |

---

## 狀態管理

### 全域變數

```javascript
// 核心元件
let renderer, scene, camera, stats;
let pointclouds;  // Points[] - 三組點雲陣列
let raycaster;

// 互動狀態
let intersection = null;  // 目前的交集結果
let spheresIndex = 0;     // 下一個要使用的球體索引
let clock;                // 時間追蹤
let toggle = 0;           // 標記生成節流

// 常數
const pointer = new THREE.Vector2();
const spheres = [];       // 球體物件池

// 配置參數
const threshold = 0.1;    // Raycaster 偵測閾值
const pointSize = 0.05;   // 點的大小
const width = 80;         // 點雲寬度
const length = 160;       // 點雲長度
const rotateY = new THREE.Matrix4().makeRotationY(0.005);
```

### 狀態轉換

```
[初始化] → [渲染迴圈開始]
              │
              ▼
    ┌─────────────────────┐
    │   每幀渲染 (render)  │◄─────────────────┐
    └─────────────────────┘                   │
              │                               │
              ▼                               │
    ┌─────────────────────┐                   │
    │  攝影機旋轉         │                   │
    │  camera.applyMatrix4│                   │
    └─────────────────────┘                   │
              │                               │
              ▼                               │
    ┌─────────────────────┐                   │
    │  Raycasting 偵測    │                   │
    │  raycaster.setFrom  │                   │
    │  Camera(pointer,    │                   │
    │  camera)            │                   │
    └─────────────────────┘                   │
              │                               │
              ▼                               │
    ┌─────────────────────┐                   │
    │  有交集且 toggle    │                   │
    │  > 0.02?           │                   │
    └─────────────────────┘                   │
         │YES      │NO                        │
         ▼         │                          │
    ┌────────────┐ │                          │
    │ 放置新標記  │ │                          │
    │ spheres[   │ │                          │
    │ spheres    │ │                          │
    │ Index]     │ │                          │
    └────────────┘ │                          │
         │         │                          │
         ▼         ▼                          │
    ┌─────────────────────┐                   │
    │  所有球體縮放       │                   │
    │  scale *= 0.98     │                   │
    └─────────────────────┘                   │
              │                               │
              ▼                               │
    ┌─────────────────────┐                   │
    │  toggle +=          │                   │
    │  clock.getDelta()   │                   │
    └─────────────────────┘                   │
              │                               │
              ▼                               │
    ┌─────────────────────┐                   │
    │  renderer.render    │───────────────────┘
    │  (scene, camera)    │
    └─────────────────────┘
```

---

## 數學公式

### 點雲波浪形狀

```
y = (cos(u × π × 4) + sin(v × π × 8)) / 20

其中:
- u = i / width (0 ~ 1)
- v = j / length (0 ~ 1)
- x = u - 0.5 (-0.5 ~ 0.5)
- z = v - 0.5 (-0.5 ~ 0.5)
```

### 顏色強度

```
intensity = (y + 0.1) × 5

其中:
- y 為點的 Y 座標
- 結果用於乘以基礎顏色
```

### 滑鼠座標轉換

```
pointer.x = (clientX / innerWidth) × 2 - 1
pointer.y = -(clientY / innerHeight) × 2 + 1
```

### 球體縮放

```
newScale = currentScale × 0.98
finalScale = clamp(newScale, 0.01, 1)
```
