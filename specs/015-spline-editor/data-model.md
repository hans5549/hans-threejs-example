# Data Model: WebGL Geometry Spline Editor

**Feature**: `015-spline-editor`  
**Date**: 2025-12-04  
**Status**: Complete

## 概述

本文件定義 Spline Editor 的核心資料模型，包括實體、屬性和關係。

---

## 核心實體

### 1. ControlPoint (控制點)

代表曲線上的可編輯節點。

| 屬性 | 類型 | 說明 |
|------|------|------|
| `mesh` | `THREE.Mesh` | 視覺化的立方體物件 |
| `position` | `THREE.Vector3` | 3D 空間座標 (x, y, z) |
| `material` | `THREE.MeshLambertMaterial` | 材質（隨機顏色） |

**行為**:
- 可被 Raycaster 選取
- 可被 TransformControls 拖曳
- 位置變更時觸發曲線更新

**約束**:
- 最少 4 個控制點
- 立方體尺寸: 20 × 20 × 20

---

### 2. Spline (曲線)

由控制點定義的 Catmull-Rom 曲線。

| 屬性 | 類型 | 說明 |
|------|------|------|
| `curve` | `THREE.CatmullRomCurve3` | 曲線物件 |
| `curveType` | `string` | 曲線類型: 'catmullrom' \| 'centripetal' \| 'chordal' |
| `tension` | `number` | 張力參數 (0-1)，僅對 catmullrom 有效 |
| `mesh` | `THREE.Line` | 渲染用的線條物件 |
| `color` | `number` | 線條顏色 |
| `visible` | `boolean` | 是否顯示 |

**曲線類型對照**:

| 類型 | curveType | 顏色 | 說明 |
|------|-----------|------|------|
| Uniform | `catmullrom` | 紅色 (0xff0000) | 可調整 tension |
| Centripetal | `centripetal` | 綠色 (0x00ff00) | 避免尖點 |
| Chordal | `chordal` | 藍色 (0x0000ff) | 基於弦長 |

**約束**:
- 弧段數量 (ARC_SEGMENTS): 200

---

### 3. Scene (場景)

3D 環境容器。

| 組件 | 類型 | 說明 |
|------|------|------|
| `camera` | `THREE.PerspectiveCamera` | 透視相機 |
| `scene` | `THREE.Scene` | 場景容器 |
| `renderer` | `THREE.WebGLRenderer` | WebGL 渲染器 |
| `ambientLight` | `THREE.AmbientLight` | 環境光 |
| `spotLight` | `THREE.SpotLight` | 聚光燈（帶陰影） |
| `plane` | `THREE.Mesh` | 地面平面 |
| `gridHelper` | `THREE.GridHelper` | 網格輔助線 |

**相機參數**:
- FOV: 70°
- Near: 1
- Far: 10000
- 初始位置: (0, 250, 1000)

**地面參數**:
- 尺寸: 2000 × 2000
- Y 位置: -200

---

### 4. Controls (控制器)

互動控制元件。

| 控制器 | 類型 | 說明 |
|--------|------|------|
| `orbitControls` | `OrbitControls` | 軌道相機控制 |
| `transformControl` | `TransformControls` | 物件變換控制 |

**協調規則**:
- TransformControls 拖曳時，OrbitControls 停用
- 點擊控制點時，TransformControls 綁定到該物件

---

### 5. GUI (控制面板)

使用者介面控制。

| 控制項 | 類型 | 預設值 | 說明 |
|--------|------|--------|------|
| `uniform` | checkbox | `true` | 顯示/隱藏 uniform 曲線 |
| `tension` | slider | `0.5` | uniform 曲線張力 (0-1) |
| `centripetal` | checkbox | `true` | 顯示/隱藏 centripetal 曲線 |
| `chordal` | checkbox | `true` | 顯示/隱藏 chordal 曲線 |
| `addPoint` | button | - | 新增控制點 |
| `removePoint` | button | - | 移除最後一個控制點 |
| `exportSpline` | button | - | 匯出控制點座標 |

---

## 資料流

```text
┌─────────────────┐
│   User Input    │
│ (Mouse/GUI)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Raycaster     │ ─── 點擊選取 ──▶ TransformControls.attach()
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│TransformControls│ ─── 拖曳 ──▶ ControlPoint.position 更新
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Spline[]      │ ─── 更新 ──▶ curve.getPoints() → mesh.geometry
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    Renderer     │ ─── 渲染 ──▶ 畫面輸出
└─────────────────┘
```

---

## 初始狀態

### 預設控制點座標

```javascript
const defaultPositions = [
  new THREE.Vector3(289.76843686945404, 452.51481137238443, 56.10018915737797),
  new THREE.Vector3(-53.56300074753207, 171.49711742836848, -14.495472686253045),
  new THREE.Vector3(-91.40118730204415, 176.4306956436485, -6.958271935582161),
  new THREE.Vector3(-383.785318791128, 491.1365363371675, 47.869296953772746)
];
```

---

## 狀態管理

### 全域狀態變數

| 變數 | 類型 | 說明 |
|------|------|------|
| `splineHelperObjects` | `THREE.Mesh[]` | 控制點物件陣列 |
| `positions` | `THREE.Vector3[]` | 控制點位置陣列 |
| `splinePointsLength` | `number` | 控制點數量 |
| `splines` | `Object` | 曲線物件集合 |
| `transformControl` | `TransformControls` | 當前變換控制器 |

### 狀態變更事件

| 事件 | 觸發條件 | 處理方式 |
|------|----------|----------|
| 控制點新增 | GUI addPoint | push to arrays, updateSplineOutline() |
| 控制點移除 | GUI removePoint | pop from arrays, updateSplineOutline() |
| 控制點移動 | TransformControls objectChange | updateSplineOutline() |
| 曲線可見性 | GUI checkbox | splines[k].mesh.visible = value |
| 曲線張力 | GUI tension slider | splines.uniform.tension = value |
