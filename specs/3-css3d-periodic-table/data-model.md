# Data Model: CSS3D 週期表

**Feature**: 3-css3d-periodic-table  
**Date**: 2025-11-28

---

## Entity Definitions

### 1. Chemical Element (化學元素)

化學週期表中的每個元素，包含顯示所需的基本資訊。

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `symbol` | string | 化學符號 (1-3 字元) | "H", "He", "Li" |
| `name` | string | 元素英文名稱 | "Hydrogen", "Helium" |
| `atomicMass` | string | 原子量 (含括號表示不穩定) | "1.00794", "(237)" |
| `column` | number | 週期表欄位 (1-18) | 1, 18, 13 |
| `row` | number | 週期表列 (1-10, 含鑭系錒系) | 1, 2, 9 |

**Storage Format**: 扁平陣列，每 5 個值為一組

```javascript
const table = [
    // symbol, name, atomicMass, column, row
    'H', 'Hydrogen', '1.00794', 1, 1,
    'He', 'Helium', '4.002602', 18, 1,
    // ...
];
```

**Validation Rules**:
- symbol: 非空字串，1-3 字元
- name: 非空字串
- atomicMass: 非空字串
- column: 整數 1-18
- row: 整數 1-10

**Total Elements**: 118 個 (原子序 1-118)

---

### 2. CSS3D Object (3D 物件)

每個化學元素在 3D 場景中的視覺呈現物件。

| Field | Type | Description |
|-------|------|-------------|
| `element` | HTMLElement | 對應的 DOM 元素 (div.element) |
| `position` | Vector3 | 當前 3D 位置 (x, y, z) |
| `rotation` | Euler | 當前旋轉角度 (x, y, z) |
| `atomicNumber` | number | 原子序 (用於識別) |

**DOM Structure**:
```html
<div class="element">
    <div class="number">1</div>
    <div class="symbol">H</div>
    <div class="details">Hydrogen<br>1.00794</div>
</div>
```

**Relationships**:
- 1:1 對應 Chemical Element
- 儲存於 `objects[]` 陣列中

---

### 3. Target Layout (目標佈局)

定義四種視圖中每個元素的目標位置與旋轉。

| Field | Type | Description |
|-------|------|-------------|
| `position` | Vector3 | 目標位置 (x, y, z) |
| `rotation` | Euler | 目標旋轉 (x, y, z) |

**Layout Types**:

```javascript
const targets = {
    table: [],   // 118 個 Object3D - 週期表平面佈局
    sphere: [],  // 118 個 Object3D - 球形分布
    helix: [],   // 118 個 Object3D - 螺旋分布
    grid: []     // 118 個 Object3D - 3D 網格分布
};
```

---

## Layout Algorithms

### Table Layout

```
Position Formula:
  x = (column × 140) - 1330
  y = -(row × 180) + 990
  z = 0

Visual Result: 標準週期表的 18 欄 × 10 列佈局
```

### Sphere Layout

```
Position Formula (Spherical Coordinates):
  φ (phi) = acos(-1 + 2i/n)     // 極角
  θ (theta) = √(n × π) × φ      // 方位角
  r = 800                        // 半徑

  x = r × sin(φ) × cos(θ)
  y = r × cos(φ)
  z = r × sin(φ) × sin(θ)

Rotation: lookAt(center × 2)     // 面向球心外側

Visual Result: 球面均勻分布的點
```

### Helix Layout

```
Position Formula (Cylindrical Coordinates):
  θ = i × 0.175 + π             // 角度
  r = 900                        // 半徑
  y = -(i × 8) + 450            // 垂直位置

  x = r × cos(θ)
  z = r × sin(θ)

Rotation: lookAt(x×2, y, z×2)   // 面向外側

Visual Result: 螺旋下降的卡片排列
```

### Grid Layout

```
Position Formula:
  x = ((i % 5) × 400) - 800          // 5 個一行
  y = (-(⌊i/5⌋ % 5) × 400) + 800     // 5 行一層
  z = (⌊i/25⌋) × 1000 - 2000         // 共 5 層

Visual Result: 5×5×5 的立方網格 (最後一層不滿)
```

---

## State Transitions

```
┌──────────────────────────────────────────────────┐
│                   Page Load                       │
│  objects[]: random positions                     │
└─────────────────────┬────────────────────────────┘
                      │ auto transform(targets.table)
                      ▼
┌──────────────────────────────────────────────────┐
│                 Table View                        │
│  objects[].position = targets.table[].position   │
└─────────────────────┬────────────────────────────┘
          │           │           │           │
    click TABLE  click SPHERE  click HELIX  click GRID
          │           │           │           │
          ▼           ▼           ▼           ▼
       Table       Sphere       Helix        Grid
        View        View        View         View
```

**Transition Duration**: 2000-4000ms (隨機 + 基礎值)
**Easing**: TWEEN.Easing.Exponential.InOut

---

## Complete Element Data

完整的 118 個化學元素資料：

```javascript
const table = [
    'H', 'Hydrogen', '1.00794', 1, 1,
    'He', 'Helium', '4.002602', 18, 1,
    'Li', 'Lithium', '6.941', 1, 2,
    'Be', 'Beryllium', '9.012182', 2, 2,
    'B', 'Boron', '10.811', 13, 2,
    'C', 'Carbon', '12.0107', 14, 2,
    'N', 'Nitrogen', '14.0067', 15, 2,
    'O', 'Oxygen', '15.9994', 16, 2,
    'F', 'Fluorine', '18.9984032', 17, 2,
    'Ne', 'Neon', '20.1797', 18, 2,
    // ... (完整資料見實作檔案)
];
```

---

## CSS Styling Model

### Element Card

| Property | Value | Purpose |
|----------|-------|---------|
| width | 120px | 卡片寬度 |
| height | 160px | 卡片高度 |
| background | rgba(0,127,127, 0.25-0.75) | 隨機透明度青色 |
| border | 1px solid rgba(127,255,255,0.25) | 青色邊框 |
| box-shadow | 0 0 12px rgba(0,255,255,0.5) | 發光效果 |

### Hover State

| Property | Value |
|----------|-------|
| border | 1px solid rgba(127,255,255,0.75) |
| box-shadow | 0 0 12px rgba(0,255,255,0.75) |

### Sub-elements

| Element | Style |
|---------|-------|
| .number | top-right, 12px, cyan |
| .symbol | center, 60px bold, white with glow |
| .details | bottom, 12px, cyan |
