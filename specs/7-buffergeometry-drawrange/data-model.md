# Data Model: BufferGeometry DrawRange 互動式粒子網絡

**Date**: 2025-11-28  
**Source**: [spec.md](spec.md)

---

## 核心實體

### 1. ParticleData

代表單一粒子的運動狀態資料。

| 屬性 | 類型 | 說明 | 驗證規則 |
|------|------|------|----------|
| velocity | THREE.Vector3 | 速度向量 | 各分量範圍 [-1, 1] |
| numConnections | number | 當前連接數量 | ≥0，每幀重置 |

**狀態轉換**:
- 初始化：隨機速度向量
- 每幀更新：根據邊界反彈重新計算速度方向
- 連接計算：累加 numConnections 直到達到 maxConnections 或幀結束

---

### 2. EffectController

儲存所有 GUI 可調整的參數。

| 屬性 | 類型 | 預設值 | 範圍 | 說明 |
|------|------|--------|------|------|
| showDots | boolean | true | - | 顯示/隱藏粒子 |
| showLines | boolean | true | - | 顯示/隱藏連線 |
| minDistance | number | 150 | 10-300 | 連線生成距離門檻 |
| limitConnections | boolean | false | - | 是否限制連接數量 |
| maxConnections | number | 20 | 0-30 | 每粒子最大連接數 |
| particleCount | number | 500 | 0-1000 | 顯示的粒子數量 |

---

### 3. BufferGeometry 資料結構

#### ParticlePositions（粒子位置緩衝區）

| 屬性 | 類型 | 大小 | 說明 |
|------|------|------|------|
| position | Float32Array | maxParticleCount × 3 | x, y, z 座標 |
| usage | THREE.Usage | DynamicDrawUsage | 動態更新模式 |
| drawRange | [start, count] | [0, particleCount] | 實際渲染數量 |

#### LinePositions（連線頂點緩衝區）

| 屬性 | 類型 | 大小 | 說明 |
|------|------|------|------|
| position | Float32Array | segments × 3 | 線段端點座標 |
| color | Float32Array | segments × 3 | RGB 透明度（使用相同值表示灰階） |
| usage | THREE.Usage | DynamicDrawUsage | 動態更新模式 |
| drawRange | [start, count] | [0, numConnected × 2] | 實際渲染頂點數 |

**segments 計算**: maxParticleCount × maxParticleCount（最壞情況）

---

## 常數配置

| 常數 | 值 | 說明 |
|------|-----|------|
| maxParticleCount | 1000 | 最大粒子數量 |
| r | 800 | 立方體邊界大小 |
| rHalf | 400 | 邊界半徑 |
| cameraZ | 1750 | 攝影機初始 Z 位置 |
| cameraFOV | 45 | 視野角度（度） |
| controlsMinDistance | 1000 | 攝影機最小距離 |
| controlsMaxDistance | 3000 | 攝影機最大距離 |

---

## 關係圖

```
┌─────────────────────────────────────────────────────────────┐
│                         Scene                                │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                        Group                           │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐   │  │
│  │  │  BoxHelper  │  │  PointCloud │  │  LinesMesh   │   │  │
│  │  │  (邊界框)   │  │  (粒子系統) │  │  (連線網絡)  │   │  │
│  │  └─────────────┘  └──────┬──────┘  └──────┬───────┘   │  │
│  │                          │                 │           │  │
│  │                  ┌───────┴─────┐   ┌──────┴──────┐    │  │
│  │                  │ particles   │   │  geometry   │    │  │
│  │                  │ (Geometry)  │   │ (Geometry)  │    │  │
│  │                  └───────┬─────┘   └──────┬──────┘    │  │
│  │                          │                 │           │  │
│  │                  ┌───────┴─────┐   ┌──────┴──────┐    │  │
│  │                  │ position    │   │ position    │    │  │
│  │                  │ attribute   │   │ color       │    │  │
│  │                  │             │   │ attributes  │    │  │
│  │                  └─────────────┘   └─────────────┘    │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌─────────────────┐  ┌─────────────────────────────────┐   │
│  │ OrbitControls   │  │  particlesData[]                │   │
│  │ (攝影機控制)    │  │  [{ velocity, numConnections }] │   │
│  └─────────────────┘  └─────────────────────────────────┘   │
│                                                              │
│  ┌─────────────────┐  ┌─────────────────────────────────┐   │
│  │ GUI (lil-gui)   │  │  effectController               │   │
│  │ (控制面板)      │  │  { showDots, minDistance, ... } │   │
│  └────────┬────────┘  └─────────────────────────────────┘   │
│           │                           ▲                      │
│           └───────────────────────────┘                      │
│                  onChange 事件綁定                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 資料流程

### 初始化流程

```
1. 建立 particlePositions Float32Array
2. 為每個粒子 (0 to maxParticleCount):
   a. 隨機生成 x, y, z 座標 (範圍: -rHalf to rHalf)
   b. 儲存到 particlePositions[i*3], [i*3+1], [i*3+2]
   c. 建立 particleData { velocity: random, numConnections: 0 }
   d. 推入 particlesData 陣列
3. 建立 BufferGeometry 並設定 position attribute
4. 設定 drawRange(0, particleCount)
```

### 每幀更新流程

```
1. 重置所有 particleData.numConnections = 0
2. 更新粒子位置:
   for i = 0 to particleCount:
     a. 位置 += 速度
     b. 邊界檢查 → 反轉速度
3. 計算連線:
   numConnected = 0
   for i = 0 to particleCount:
     for j = i+1 to particleCount:
       a. 計算距離
       b. if 距離 < minDistance:
          - 檢查連接數量限制
          - 計算 alpha = 1 - 距離/minDistance
          - 寫入線段頂點和顏色
          - numConnected++
4. 更新 geometry:
   a. linesMesh.geometry.setDrawRange(0, numConnected * 2)
   b. 設定 needsUpdate = true
5. 渲染場景
```
