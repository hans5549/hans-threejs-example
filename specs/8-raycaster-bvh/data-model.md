# Data Model: WebGL Raycaster BVH

**Feature**: `8-raycaster-bvh`  
**Date**: 2025-01-XX  
**Status**: In Design

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
│   Model Mesh    │  │   Ray Origins   │  │  Intersection   │
│ (Stanford Bunny)│  │ (InstancedMesh) │  │     Points      │
│  - geometry     │  │  - count: 100   │  │ (InstancedMesh) │
│  - boundsTree   │  │  - geometry:    │  │  - count: 動態  │
│  - material     │  │    SphereGeom   │  │  - geometry:    │
└─────────────────┘  └─────────────────┘  │    SphereGeom   │
         │                                 └─────────────────┘
         │                                          ▲
         ▼                                          │
┌─────────────────┐                                │
│    MeshBVH      │                                │
│  - rootData     │                                │
│  - indices      │                                │
│  - maxDepth: 40 │                                │
└─────────────────┘                                │
         │                                          │
         │                                          │
         ▼                                          │
┌─────────────────────────────────────────────────────────────────┐
│                        Raycaster                                 │
│  - firstHitOnly: false                                           │
│  - origin: Ray Origin Points[i]                                  │
│  - direction: 朝向模型中心的單位向量                             │
│  - threshold: 預設 (0)                                           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ raycast()
                                ▼
                    ┌─────────────────────┐
                    │    Intersection     │────────────────────┘
                    │  - point: Vector3   │
                    │  - distance: number │
                    │  - object: Mesh     │
                    └─────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       BVH Helper                                 │
│  (可選的 BVH 視覺化)                                             │
│  - mesh: 目標網格                                                │
│  - depth: 顯示深度 (預設: 10)                                    │
│  - color: 邊界框顏色                                             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   Performance Monitor                            │
│  - fpsHistory: number[]                                          │
│  - avgFPS: number                                                │
│  - warningActive: boolean                                        │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     Loading State                                │
│  - status: 'idle' | 'loading' | 'building' | 'ready' | 'error'  │
│  - progress: 0-100 (模型載入百分比)                              │
│  - message: string (狀態訊息)                                    │
│  - startTime: number (開始時間戳記)                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 核心實體定義

### 1. Scene (場景)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `children` | Object3D[] | 場景中的所有物件 |
| `background` | Color | 背景顏色 (預設: 0x000000) |
| `fog` | Fog \| null | 霧效 (可選) |

**關聯物件：**
- Model Mesh (Stanford Bunny)
- Ray Origins (InstancedMesh)
- Intersection Points (InstancedMesh)
- Ray Lines (Line[])
- BVH Helper (可選)

---

### 2. Camera (攝影機)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `fov` | number | 視野角度: 60° |
| `aspect` | number | 長寬比: window.innerWidth / window.innerHeight |
| `near` | number | 近裁剪面: 0.1 |
| `far` | number | 遠裁剪面: 100 |
| `position` | Vector3 | 初始位置: (10, 10, 10) |

**OrbitControls 設定：**
- `enableDamping`: true
- `dampingFactor`: 0.05
- `target`: (0, 0, 0) - 模型中心

---

### 3. ModelMesh (模型網格)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `geometry` | BufferGeometry | FBX 模型的幾何資料 |
| `material` | MeshPhongMaterial | 標準光照材質 |
| `position` | Vector3 | 世界座標位置: (0, 0, 0) |
| `scale` | Vector3 | 縮放比例 (依模型調整) |
| `castShadow` | boolean | 投射陰影: false |
| `receiveShadow` | boolean | 接收陰影: false |

#### BufferGeometry (Stanford Bunny)

| Attribute | 類型 | 元素大小 | 說明 |
|-----------|------|----------|------|
| `position` | Float32Array | 3 | 每個頂點的 (x, y, z) 座標 |
| `normal` | Float32Array | 3 | 每個頂點的法線向量 |
| `uv` | Float32Array | 2 | 紋理座標 (可選) |
| `index` | Uint32Array | 1 | 三角形索引陣列 (~70k triangles) |

**boundsTree 屬性：**
- 類型: `MeshBVH`
- 用途: 加速光線追蹤查詢

---

### 4. MeshBVH (邊界體積階層)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `geometry` | BufferGeometry | 關聯的幾何體 |
| `_roots` | Array | BVH 樹的根節點陣列 |
| `_indirectBuffer` | boolean | 是否使用間接緩衝區 |

**建構選項：**
```typescript
{
    strategy: 'CENTER',    // 分割策略 (CENTER | AVERAGE | SAH)
    maxDepth: 40,          // 最大樹深度
    maxLeafTris: 10,       // 葉節點最大三角形數量
    setBoundingBox: true,  // 自動設定邊界框
    verbose: false         // 不顯示警告訊息
}
```

**關鍵方法：**
- `raycastFirst(ray)`: 回傳第一個交點
- `raycast(ray)`: 回傳所有交點（未排序）
- `refit()`: 重新調整邊界框（頂點變更後）

---

### 5. RayOrigins (光線原點 - InstancedMesh)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `geometry` | SphereGeometry | 球體幾何: radius=0.03, segments=16 |
| `material` | MeshBasicMaterial | 基礎材質: color=0x00ff00, opacity=0.6 |
| `count` | number | 實例數量: 100 (可調整) |
| `instanceMatrix` | InstancedBufferAttribute | 實例變換矩陣 |

**分布演算法：**
- 使用 Fibonacci 球面採樣
- 均勻分布在半徑 5 的球面上
- 朝向模型中心 (0, 0, 0)

**矩陣更新：**
```javascript
origins.forEach((position, index) => {
    matrix.setPosition(position);
    rayOriginsMesh.setMatrixAt(index, matrix);
});
rayOriginsMesh.instanceMatrix.needsUpdate = true;
```

---

### 6. IntersectionPoints (交點標記 - InstancedMesh)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `geometry` | SphereGeometry | 球體幾何: radius=0.05, segments=8 |
| `material` | MeshBasicMaterial | 基礎材質: color=0xff0000 |
| `count` | number | 實例數量: 動態 (0-100) |
| `instanceMatrix` | InstancedBufferAttribute | 實例變換矩陣 |

**更新機制：**
- 每次光線追蹤後更新
- `count` 屬性設定為實際交點數量
- 未使用的實例不會被渲染

---

### 7. RayLine (光線視覺化)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `geometry` | BufferGeometry | 兩點連線: origin → intersection |
| `material` | LineBasicMaterial | 線條材質 |

#### LineBasicMaterial 設定

| 屬性 | 類型 | 值 | 說明 |
|------|------|-----|------|
| `color` | Color | 0xffffff | 白色光線 |
| `transparent` | boolean | true | 啟用透明度 |
| `opacity` | number | 0.5 | 半透明 (可調整) |
| `linewidth` | number | 1 | 線條寬度 (注意: WebGL 限制) |

**BufferGeometry 資料：**
```javascript
const points = [
    rayOrigin.clone(),
    intersectionPoint.clone()
];
const geometry = new THREE.BufferGeometry().setFromPoints(points);
```

---

### 8. BVHHelper (BVH 視覺化輔助器)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `mesh` | Mesh | 目標網格 (Stanford Bunny) |
| `depth` | number | 顯示深度: 10 (可調整 1-20) |
| `color` | Color | 邊界框顏色: 0x00FF88 |
| `opacity` | number | 透明度: 0.3 |
| `displayParents` | boolean | 顯示父節點: false |
| `displayEdges` | boolean | 顯示邊緣: true |

**更新觸發：**
- `depth` 參數變更時
- BVH 重建後
- 手動呼叫 `update()` 方法

---

### 9. PerformanceMonitor (性能監控器)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `fpsHistory` | number[] | FPS 歷史記錄 (60 幀) |
| `avgFPS` | number | 平均 FPS |
| `warningThreshold` | number | 警告閾值: 30 |
| `warningActive` | boolean | 警告狀態 |
| `warningElement` | HTMLDivElement | 警告 UI 元素 |

**計算邏輯：**
```javascript
update(deltaTime) {
    const fps = 1 / deltaTime;
    this.fpsHistory.push(fps);
    if (this.fpsHistory.length > 60) {
        this.fpsHistory.shift();
    }
    this.avgFPS = this.fpsHistory.reduce((a, b) => a + b, 0) / this.fpsHistory.length;
    
    if (this.avgFPS < 30 && !this.warningActive) {
        this.showWarning();
    } else if (this.avgFPS >= 30 && this.warningActive) {
        this.hideWarning();
    }
}
```

---

### 10. LoadingState (載入狀態)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `status` | LoadingStatus | 當前狀態 |
| `progress` | number | 載入進度: 0-100 |
| `message` | string | 狀態訊息 |
| `startTime` | number | 開始時間戳記 (performance.now()) |
| `timeoutId` | number | 超時計時器 ID |

#### LoadingStatus 枚舉

```typescript
type LoadingStatus = 
    | 'idle'       // 未開始
    | 'loading'    // 載入 FBX 模型
    | 'building'   // 建構 BVH
    | 'ready'      // 準備完成
    | 'error'      // 發生錯誤
    | 'timeout';   // 載入超時
```

**狀態轉換：**
```
idle → loading → building → ready
       ↓            ↓
     error      timeout
```

**UI 元素：**
- `progressBar`: 載入進度條
- `progressText`: 進度百分比文字
- `statusMessage`: 狀態描述訊息

---

### 11. GUIParams (GUI 控制參數)

| 屬性 | 類型 | 預設值 | 範圍 | 說明 |
|------|------|--------|------|------|
| `rayCount` | number | 100 | 1-200 | 光線數量 |
| `enableBVH` | boolean | true | - | 啟用 BVH 加速 |
| `showBVHHelper` | boolean | false | - | 顯示 BVH 視覺化 |
| `helperDepth` | number | 10 | 1-20 | BVH Helper 深度 |
| `rayOpacity` | number | 0.5 | 0-1 | 光線透明度 |
| `animate` | boolean | true | - | 自動旋轉 |

**回呼函式對應：**
- `rayCount` → `updateRays()`
- `enableBVH` → `toggleBVH()`
- `showBVHHelper` → `toggleBVHHelper()`
- `helperDepth` → `updateBVHHelper()`
- `rayOpacity` → `updateRayMaterial()`

---

## 資料流與狀態轉換

### 初始化流程

```
1. 建立場景、攝影機、渲染器
   ↓
2. 建立 LoadingState (status: 'idle')
   ↓
3. 開始載入 FBX 模型 (status: 'loading')
   ↓ (監聽 LoadingManager 進度)
   ↓
4. 模型載入完成
   ↓
5. 建構 BVH (status: 'building')
   ↓ (測量建構時間)
   ↓
6. 初始化完成 (status: 'ready')
   ↓
7. 產生光線原點 (Fibonacci 球面採樣)
   ↓
8. 執行光線追蹤
   ↓
9. 視覺化結果 (光線、交點)
   ↓
10. 進入渲染循環
```

### 互動流程（GUI 控制）

```
使用者調整 GUI 參數
   ↓
onChange 回呼觸發
   ↓
┌───────────────────┬───────────────────┬───────────────────┐
│   rayCount 變更   │   enableBVH 變更  │ showBVHHelper 變更│
│                   │                   │                   │
│ 1. 清除舊光線     │ 1. 切換 raycast   │ 1. 建立/隱藏      │
│ 2. 產生新原點     │    函式           │    BVHHelper      │
│ 3. 執行光線追蹤   │ 2. 重新執行光線   │ 2. 更新深度       │
│ 4. 更新視覺化     │    追蹤           │ 3. 重新渲染       │
└───────────────────┴───────────────────┴───────────────────┘
                       ↓
              更新 PerformanceMonitor
                       ↓
              檢查 FPS 是否 < 30
                       ↓
            (是) → 顯示警告 UI
            (否) → 隱藏警告 UI
```

### 錯誤處理流程

```
模型載入/BVH 建構失敗
   ↓
LoadingState.status = 'error'
   ↓
顯示錯誤訊息 UI
   ↓
提供「重新載入」按鈕
   ↓
使用者點擊 → 重新執行初始化流程
```

---

## 性能考量

### 記憶體佔用估算

| 組件 | 估計大小 | 說明 |
|------|---------|------|
| FBX 模型 (70k tris) | ~15 MB | 頂點、法線、索引資料 |
| MeshBVH | ~3 MB | 樹結構 + 索引緩衝區 |
| Ray Origins (100) | ~0.5 MB | InstancedMesh 矩陣 |
| Intersection Points (100) | ~0.5 MB | InstancedMesh 矩陣 |
| Ray Lines (100) | ~1 MB | BufferGeometry 陣列 |
| 紋理/材質 | ~5 MB | 模型材質資源 |
| **總計** | **~25 MB** | 瀏覽器可接受範圍 |

### 關鍵性能指標

| 指標 | 目標 | 測量方法 |
|-----|------|---------|
| FPS (BVH 啟用) | ≥60 | Stats.js |
| FPS (BVH 停用) | 10-20 | Stats.js |
| BVH 建構時間 | <500ms | performance.now() |
| 單幀光線追蹤 (100 rays) | <2ms | performance.now() |
| BVH 切換回應時間 | <100ms | performance.now() |

---

## 驗證規則

### 資料完整性檢查

1. **模型載入驗證：**
   - `geometry.attributes.position` 存在
   - `geometry.index` 存在
   - 三角形數量 > 0

2. **BVH 建構驗證：**
   - `geometry.boundsTree` 存在
   - `boundsTree._roots` 非空
   - 無建構錯誤

3. **光線追蹤驗證：**
   - 所有 ray origins 為有效的 Vector3
   - ray directions 為單位向量
   - intersection points 在模型邊界框內

### 邊界條件處理

| 情境 | 處理方式 |
|------|---------|
| 模型載入超時 (>60s) | 顯示錯誤訊息 + 重試選項 |
| BVH 建構失敗 | 回退到標準 raycasting |
| 光線數量 = 0 | 清除所有視覺化元素 |
| FPS < 30 持續 1 秒 | 顯示性能警告 |
| 視窗大小變更 | 更新攝影機 aspect + 渲染器大小 |

---

## 參考資料

- [THREE.BufferGeometry](https://threejs.org/docs/#api/en/core/BufferGeometry)
- [THREE.InstancedMesh](https://threejs.org/docs/#api/en/objects/InstancedMesh)
- [THREE.Raycaster](https://threejs.org/docs/#api/en/core/Raycaster)
- [three-mesh-bvh MeshBVH](https://github.com/gkjohnson/three-mesh-bvh#meshbvh)
- [three-mesh-bvh MeshBVHHelper](https://github.com/gkjohnson/three-mesh-bvh#meshbvhhelper)
