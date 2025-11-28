# Component Interface: CSS3D 週期表

**Feature**: 3-css3d-periodic-table  
**Date**: 2025-11-28

---

## Overview

此文件定義 CSS3D 週期表應用程式的元件介面規格。由於本專案為純前端靜態展示應用，無後端 API，此處定義的是 JavaScript 模組內部的函式介面。

---

## Core Functions Interface

### init()

初始化 3D 場景和所有元件。

```typescript
function init(): void
```

**Side Effects**:
- 建立 PerspectiveCamera
- 建立 THREE.Scene
- 建立 118 個 CSS3DObject 並加入場景
- 計算四種佈局的目標位置
- 建立 CSS3DRenderer 並掛載到 #container
- 建立 TrackballControls
- 綁定按鈕事件監聽器
- 觸發初始動畫 (transform to table)
- 綁定 window resize 事件
- 開始 animate() 迴圈

---

### transform(targets, duration)

執行視圖切換動畫。

```typescript
function transform(
    targets: THREE.Object3D[],  // 目標位置陣列 (長度 118)
    duration: number            // 基礎動畫時間 (毫秒)
): void
```

**Parameters**:
| Name | Type | Description |
|------|------|-------------|
| targets | Object3D[] | 118 個目標位置物件，每個包含 position 和 rotation |
| duration | number | 基礎動畫時間，實際時間為 duration + random(0, duration) |

**Behavior**:
1. 移除所有現有動畫 (TWEEN.removeAll())
2. 為每個物件建立位置動畫
3. 為每個物件建立旋轉動畫
4. 建立渲染更新動畫

---

### onWindowResize()

處理視窗大小調整。

```typescript
function onWindowResize(): void
```

**Behavior**:
1. 更新 camera.aspect
2. 呼叫 camera.updateProjectionMatrix()
3. 更新 renderer.setSize()
4. 呼叫 render()

---

### animate()

主動畫迴圈。

```typescript
function animate(): void
```

**Behavior**:
1. requestAnimationFrame(animate)
2. TWEEN.update()
3. controls.update()

---

### render()

渲染場景。

```typescript
function render(): void
```

**Behavior**:
- renderer.render(scene, camera)

---

## Data Structures

### Element Data Array

```typescript
// 扁平陣列格式
const table: (string | number)[] = [
    // [symbol, name, atomicMass, column, row] × 118
    'H', 'Hydrogen', '1.00794', 1, 1,
    // ...
];

// 存取方式
for (let i = 0; i < table.length; i += 5) {
    const symbol = table[i] as string;
    const name = table[i + 1] as string;
    const atomicMass = table[i + 2] as string;
    const column = table[i + 3] as number;
    const row = table[i + 4] as number;
}
```

### Target Layouts Object

```typescript
interface TargetLayouts {
    table: THREE.Object3D[];   // 週期表佈局
    sphere: THREE.Object3D[];  // 球形佈局
    helix: THREE.Object3D[];   // 螺旋佈局
    grid: THREE.Object3D[];    // 網格佈局
}

const targets: TargetLayouts = {
    table: [],
    sphere: [],
    helix: [],
    grid: []
};
```

---

## DOM Structure Interface

### HTML Structure

```html
<!DOCTYPE html>
<html>
<head>
    <title>three.js css3d - periodic table</title>
    <style>/* ... */</style>
</head>
<body>
    <div id="info">...</div>
    <div id="container">
        <!-- CSS3DRenderer 掛載點 -->
    </div>
    <div id="menu">
        <button id="table">TABLE</button>
        <button id="sphere">SPHERE</button>
        <button id="helix">HELIX</button>
        <button id="grid">GRID</button>
    </div>
    <script type="importmap">...</script>
    <script type="module">...</script>
</body>
</html>
```

### Element Card DOM Structure

```html
<div class="element" style="background-color: rgba(0,127,127,0.XX)">
    <div class="number">1</div>
    <div class="symbol">H</div>
    <div class="details">Hydrogen<br>1.00794</div>
</div>
```

---

## CSS Class Interface

| Class | Purpose | Required Properties |
|-------|---------|-------------------|
| `.element` | 元素卡片容器 | width, height, box-shadow, border |
| `.element:hover` | Hover 狀態 | enhanced box-shadow, border |
| `.number` | 原子序 | position: absolute, top-right |
| `.symbol` | 化學符號 | position: absolute, center, large font |
| `.details` | 元素詳情 | position: absolute, bottom |
| `button` | 視圖切換按鈕 | transparent background, cyan border |
| `button:hover` | 按鈕 hover | cyan background |
| `button:active` | 按鈕點擊 | darker background |

---

## Event Bindings

| Element | Event | Handler |
|---------|-------|---------|
| `#table` | click | transform(targets.table, 2000) |
| `#sphere` | click | transform(targets.sphere, 2000) |
| `#helix` | click | transform(targets.helix, 2000) |
| `#grid` | click | transform(targets.grid, 2000) |
| `window` | resize | onWindowResize() |
| `controls` | change | render() |
