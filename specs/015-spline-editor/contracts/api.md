# API Contracts: WebGL Geometry Spline Editor

**Feature**: `015-spline-editor`  
**Date**: 2025-12-04

## 概述

本文件定義 Spline Editor 的內部 API 契約，包括核心函式的輸入輸出規格。

---

## 核心函式

### 1. init()

初始化整個場景和控制器。

```typescript
function init(): void
```

**行為**:
- 建立 Scene、Camera、Renderer
- 設定光源和地面
- 初始化 OrbitControls 和 TransformControls
- 載入預設控制點
- 綁定事件監聽器

---

### 2. addSplineObject(position?)

新增一個控制點到場景。

```typescript
function addSplineObject(position?: THREE.Vector3): THREE.Mesh
```

**參數**:
- `position` (optional): 控制點位置，若未提供則隨機生成

**回傳**:
- 新建立的 Mesh 物件

**副作用**:
- 將 Mesh 加入 `splineHelperObjects` 陣列
- 將 Mesh 加入 Scene

---

### 3. addPoint()

透過 GUI 新增控制點。

```typescript
function addPoint(): void
```

**行為**:
- 遞增 `splinePointsLength`
- 呼叫 `addSplineObject()` 建立新控制點
- 將新位置加入 `positions` 陣列
- 呼叫 `updateSplineOutline()`
- 呼叫 `render()`

---

### 4. removePoint()

透過 GUI 移除最後一個控制點。

```typescript
function removePoint(): void
```

**行為**:
- 若 `splinePointsLength <= 4`，直接返回（保持最少 4 點）
- 從 `splineHelperObjects` 移除最後一個物件
- 遞減 `splinePointsLength`
- 從 `positions` 移除最後一個位置
- 若移除的物件正被 TransformControls 綁定，呼叫 `detach()`
- 從 Scene 移除物件
- 呼叫 `updateSplineOutline()`
- 呼叫 `render()`

---

### 5. updateSplineOutline()

更新所有曲線的幾何形狀。

```typescript
function updateSplineOutline(): void
```

**行為**:
- 遍歷 `splines` 物件
- 對每條曲線，根據控制點位置重新計算曲線上的點
- 更新 BufferGeometry 的 position attribute
- 設定 `position.needsUpdate = true`

---

### 6. exportSpline()

匯出控制點座標到主控台。

```typescript
function exportSpline(): void
```

**行為**:
- 遍歷所有控制點
- 將每個位置格式化為 `new THREE.Vector3(x, y, z)` 字串
- 輸出到 `console.log()`

---

### 7. load(newPositions)

載入一組控制點位置。

```typescript
function load(newPositions: THREE.Vector3[]): void
```

**參數**:
- `newPositions`: 控制點位置陣列

**行為**:
- 清除現有控制點（若有）
- 為每個位置呼叫 `addSplineObject(position)`
- 建立 `positions` 陣列引用
- 呼叫 `updateSplineOutline()`
- 呼叫 `render()`

---

### 8. render()

渲染場景。

```typescript
function render(): void
```

**行為**:
- 根據 `params` 設定曲線可見性
- 呼叫 `renderer.render(scene, camera)`

---

### 9. onWindowResize()

處理視窗大小變更。

```typescript
function onWindowResize(): void
```

**行為**:
- 更新 `camera.aspect`
- 呼叫 `camera.updateProjectionMatrix()`
- 呼叫 `renderer.setSize()`
- 呼叫 `render()`

---

## 事件處理

### 滑鼠事件

| 事件 | 處理函式 | 說明 |
|------|----------|------|
| `pointerdown` | `onPointerDown()` | 記錄按下位置 |
| `pointerup` | `onPointerUp()` | 判斷是否為點擊，執行選取 |
| `pointermove` | `onPointerMove()` | (可選) 用於即時回饋 |

### TransformControls 事件

| 事件 | 處理方式 |
|------|----------|
| `change` | 呼叫 `render()` |
| `dragging-changed` | 切換 OrbitControls.enabled |
| `objectChange` | 呼叫 `updateSplineOutline()` |

### OrbitControls 事件

| 事件 | 處理方式 |
|------|----------|
| `change` | 呼叫 `render()` |

---

## 狀態契約

### 全域變數

```typescript
// 控制點管理
let splineHelperObjects: THREE.Mesh[] = [];
let positions: THREE.Vector3[] = [];
let splinePointsLength: number = 4;

// 曲線管理
const splines: {
  uniform: THREE.CatmullRomCurve3;
  centripetal: THREE.CatmullRomCurve3;
  chordal: THREE.CatmullRomCurve3;
} = {};

// GUI 參數
const params: {
  uniform: boolean;
  tension: number;
  centripetal: boolean;
  chordal: boolean;
  addPoint: () => void;
  removePoint: () => void;
  exportSpline: () => void;
} = {
  uniform: true,
  tension: 0.5,
  centripetal: true,
  chordal: true,
  addPoint,
  removePoint,
  exportSpline
};
```

---

## 不變量 (Invariants)

1. `splinePointsLength >= 4` (最少 4 個控制點)
2. `splineHelperObjects.length === splinePointsLength`
3. `positions.length === splinePointsLength`
4. 每個 `splines[k].mesh.geometry.attributes.position.count === ARC_SEGMENTS`
