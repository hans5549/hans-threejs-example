# Data Model: WebGL BufferGeometry Lines

**Feature**: 11-buffergeometry-lines  
**Date**: 2025-12-01

---

## Core Entities

### 1. Application State

應用程式的全域狀態管理。

| Field | Type | Description |
|-------|------|-------------|
| container | HTMLElement | DOM 容器元素 |
| camera | PerspectiveCamera | 透視相機 |
| scene | Scene | 3D 場景 |
| renderer | WebGLRenderer | WebGL 渲染器 |
| stats | Stats | FPS 統計面板 |
| timer | Timer | 動畫計時器 |
| line | Line | 線段物件 |
| t | number | morph 動畫時間累加器 |

### 2. Geometry Configuration

幾何物件的配置常數。

| Constant | Value | Description |
|----------|-------|-------------|
| segments | 10000 | 線段頂點數量 |
| r | 800 | 分布範圍（立方體邊長） |

### 3. Camera Configuration

相機的配置參數。

| Parameter | Value | Description |
|-----------|-------|-------------|
| fov | 27 | 視野角度（度） |
| near | 1 | 近裁切平面 |
| far | 4000 | 遠裁切平面 |
| position.z | 2750 | 相機 Z 軸位置 |

### 4. Animation Parameters

動畫控制參數。

| Parameter | Formula | Description |
|-----------|---------|-------------|
| rotation.x | time × 0.25 | X 軸旋轉速率 |
| rotation.y | time × 0.5 | Y 軸旋轉速率 |
| morphInfluence | abs(sin(t)) | morph 權重（0-1 週期） |
| t increment | delta × 0.5 | morph 時間累加速率 |

---

## Data Structures

### 5. Vertex Position Array

頂點位置資料結構。

```
positions: Float32Array
├── [0]  → vertex_0.x
├── [1]  → vertex_0.y
├── [2]  → vertex_0.z
├── [3]  → vertex_1.x
├── [4]  → vertex_1.y
├── [5]  → vertex_1.z
└── ... (共 segments × 3 個元素)
```

**Generation Rule**:
```
x = Math.random() * r - r / 2   // 範圍: [-400, 400]
y = Math.random() * r - r / 2   // 範圍: [-400, 400]
z = Math.random() * r - r / 2   // 範圍: [-400, 400]
```

### 6. Vertex Color Array

頂點顏色資料結構。

```
colors: Float32Array
├── [0]  → vertex_0.r
├── [1]  → vertex_0.g
├── [2]  → vertex_0.b
├── [3]  → vertex_1.r
├── [4]  → vertex_1.g
├── [5]  → vertex_1.b
└── ... (共 segments × 3 個元素)
```

**Color Mapping Rule**:
```
r = (x / r) + 0.5   // 範圍: [0, 1]
g = (y / r) + 0.5   // 範圍: [0, 1]
b = (z / r) + 0.5   // 範圍: [0, 1]
```

### 7. Morph Target Array

Morph target 位置資料結構。

```
morphTarget: Float32BufferAttribute
├── name: "target1"
└── array: Float32Array (與 positions 結構相同)
    ├── [0]  → target_vertex_0.x
    ├── [1]  → target_vertex_0.y
    ├── [2]  → target_vertex_0.z
    └── ... (共 segments × 3 個元素)
```

---

## Object Hierarchy

### Scene Graph

```
Scene
└── Line
    ├── BufferGeometry
    │   ├── attributes
    │   │   ├── position: Float32BufferAttribute (30000 floats)
    │   │   └── color: Float32BufferAttribute (30000 floats)
    │   └── morphAttributes
    │       └── position: [Float32BufferAttribute] (1 target)
    └── LineBasicMaterial
        └── vertexColors: true
```

---

## State Transitions

### Application Lifecycle

```
[Uninitialized]
      │
      ▼ init()
[Initializing] ──(WebGL fail)──▶ [Fallback Display]
      │
      ▼ (success)
[Running] ◄────────┐
      │            │
      ▼ animate()  │
[Rendering] ───────┘
      │
      ▼ resize event
[Resizing] ────────▶ [Running]
```

### Morph Animation Cycle

```
t=0      t=π/2     t=π       t=3π/2    t=2π
│         │         │          │         │
▼         ▼         ▼          ▼         ▼
0 ──▶ 1 ──▶ 0 ──▶ 1 ──▶ 0
(original) (morphed) (original) (morphed) (original)
```

---

## Validation Rules

1. **Vertex Count**: segments MUST equal 10000
2. **Position Range**: 所有座標值 MUST 在 [-400, 400] 範圍內
3. **Color Range**: 所有顏色分量 MUST 在 [0, 1] 範圍內
4. **Morph Weight**: morphTargetInfluences[0] MUST 在 [0, 1] 範圍內
