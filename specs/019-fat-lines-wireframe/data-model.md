# Data Model: Fat Lines Wireframe

**Feature**: 019-fat-lines-wireframe  
**Date**: 2025-12-04  
**Purpose**: 定義功能中的實體、關係和資料結構

## 核心實體

### 1. Scene Graph 實體

#### Scene

- **描述**: Three.js 場景容器，包含所有 3D 物件
- **屬性**: 無自定義屬性
- **關係**: 包含 Wireframe 和 LineSegments 物件

#### Wireframe (Fat Lines 版本)

- **描述**: 使用 Fat Lines 技術渲染的線框模型
- **建立方式**: `new Wireframe(geometry, material)`
- **屬性**:
  - `geometry`: WireframeGeometry2 實例
  - `material`: LineMaterial 實例
  - `visible`: boolean - 是否顯示
- **方法**:
  - `computeLineDistances()`: 計算線段距離（虛線必需）
- **關係**: 屬於 Scene

#### LineSegments (標準 WebGL 版本)

- **描述**: 標準 WebGL 線條渲染的線框模型
- **建立方式**: `new THREE.LineSegments(geometry, material)`
- **屬性**:
  - `geometry`: WireframeGeometry 實例
  - `material`: LineBasicMaterial 或 LineDashedMaterial 實例
  - `visible`: boolean - 是否顯示
- **方法**:
  - `computeLineDistances()`: 計算線段距離（虛線必需）
- **關係**: 屬於 Scene

### 2. 幾何體實體

#### IcosahedronGeometry

- **描述**: 二十面體基礎幾何
- **建立方式**: `new THREE.IcosahedronGeometry(radius, detail)`
- **參數**:
  - `radius`: 20 (固定)
  - `detail`: 1 (細分層級)

#### WireframeGeometry2

- **描述**: Fat Lines 專用的線框幾何資料
- **建立方式**: `new WireframeGeometry2(geometry)`
- **輸入**: IcosahedronGeometry
- **輸出**: 適合 LineMaterial 使用的幾何資料

#### WireframeGeometry

- **描述**: 標準 WebGL 線框幾何資料
- **建立方式**: `new THREE.WireframeGeometry(geometry)`
- **輸入**: IcosahedronGeometry
- **輸出**: 適合 LineBasicMaterial 使用的幾何資料

### 3. 材質實體

#### LineMaterial

- **描述**: Fat Lines 專用材質，支援可變線寬
- **建立方式**: `new LineMaterial(params)`
- **屬性**:

| 屬性 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| `color` | hex | 0x4080ff | 線條顏色 |
| `linewidth` | number | 5 | 線寬（像素） |
| `dashed` | boolean | false | 虛線開關 |
| `dashScale` | number | 1 | 虛線縮放 |
| `dashSize` | number | 1 | 虛線長度 |
| `gapSize` | number | 1 | 間隔長度 |

- **特殊處理**:
  - 虛線切換需要更新 `defines.USE_DASH` 並設定 `needsUpdate = true`

#### LineBasicMaterial

- **描述**: 標準實線材質
- **建立方式**: `new THREE.LineBasicMaterial(params)`
- **屬性**:

| 屬性 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| `color` | hex | 0x4080ff | 線條顏色 |

#### LineDashedMaterial

- **描述**: 標準虛線材質
- **建立方式**: `new THREE.LineDashedMaterial(params)`
- **屬性**:

| 屬性 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| `scale` | number | 2 | 虛線縮放 |
| `dashSize` | number | 1 | 虛線長度 |
| `gapSize` | number | 1 | 間隔長度 |

### 4. 攝影機實體

#### Camera (主攝影機)

- **描述**: 主視口透視攝影機
- **建立方式**: `new THREE.PerspectiveCamera(fov, aspect, near, far)`
- **參數**:

| 參數 | 值 | 說明 |
|------|-----|------|
| `fov` | 40 | 視野角度 |
| `aspect` | window.innerWidth / window.innerHeight | 寬高比 |
| `near` | 1 | 近裁剪面 |
| `far` | 1000 | 遠裁剪面 |

- **位置**: (-50, 0, 50)

#### Camera2 (副攝影機)

- **描述**: 插入視口透視攝影機
- **建立方式**: `new THREE.PerspectiveCamera(fov, aspect, near, far)`
- **參數**:

| 參數 | 值 | 說明 |
|------|-----|------|
| `fov` | 40 | 視野角度 |
| `aspect` | 1 | 正方形 |
| `near` | 1 | 近裁剪面 |
| `far` | 1000 | 遠裁剪面 |

- **位置**: 複製自主攝影機

### 5. 控制參數實體

#### GUI Parameters

- **描述**: 使用者可調整的參數物件
- **結構**:

```javascript
{
  'line type': 0,        // 0: Fat Lines, 1: gl.LINE
  'width (px)': 5,       // 1-10
  'dashed': false,       // boolean
  'dash scale': 1,       // 0.5-1
  'dash / gap': 0        // 0: 2:1, 1: 1:1, 2: 1:2
}
```

## 實體關係圖

```
Scene
├── Wireframe (Fat Lines)
│   ├── WireframeGeometry2
│   │   └── IcosahedronGeometry
│   └── LineMaterial
│
└── LineSegments (Standard)
    ├── WireframeGeometry
    │   └── IcosahedronGeometry
    └── LineBasicMaterial / LineDashedMaterial

Cameras
├── Camera (main) ← OrbitControls
└── Camera2 (inset)

GUI
└── Parameters → 控制 LineMaterial 和材質切換
```

## 狀態轉換

### 線條類型切換

```
狀態: lineType = 0 (Fat Lines)
  → wireframe.visible = true
  → wireframe1.visible = false

狀態: lineType = 1 (gl.LINE)
  → wireframe.visible = false
  → wireframe1.visible = true
```

### 虛線切換

```
狀態: dashed = false
  → matLine.dashed = false
  → delete matLine.defines.USE_DASH
  → matLine.needsUpdate = true
  → wireframe1.material = matLineBasic

狀態: dashed = true
  → matLine.dashed = true
  → matLine.defines.USE_DASH = ''
  → matLine.needsUpdate = true
  → wireframe1.material = matLineDashed
```

## 驗證規則

| 屬性 | 規則 |
|------|------|
| linewidth | 1 ≤ value ≤ 10 |
| dashScale | 0.5 ≤ value ≤ 1 |
| dashSize | 1 或 2 (根據 dash/gap 選項) |
| gapSize | 1 或 2 (根據 dash/gap 選項) |
| minDistance | 10 |
| maxDistance | 500 |
