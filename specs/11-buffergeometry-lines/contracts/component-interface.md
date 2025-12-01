# Component Interface: WebGL BufferGeometry Lines

**Feature**: 11-buffergeometry-lines  
**Date**: 2025-12-01

---

## Module Structure

單一 HTML 檔案，所有元件以函式形式實作於 `<script type="module">` 區塊中。

---

## Core Functions

### init()

應用程式初始化入口點。

```typescript
function init(): void
```

**Responsibilities**:
1. 取得 DOM 容器元素
2. 建立透視相機並設定位置
3. 建立場景
4. 初始化 Timer 並連接 document
5. 建立 BufferGeometry 和頂點資料
6. 建立 LineBasicMaterial
7. 產生 morph targets
8. 建立 Line 物件並加入場景
9. 建立 WebGLRenderer 並設定尺寸
10. 初始化 Stats 面板
11. 註冊 resize 事件監聽器
12. 啟動動畫迴圈

**Error Handling**:
- WebGLRenderer 建立失敗時呼叫 `showFallback()`

---

### animate()

每幀動畫更新函式，由 `renderer.setAnimationLoop()` 呼叫。

```typescript
function animate(): void
```

**Responsibilities**:
1. 更新 Timer
2. 取得 delta 和 elapsed time
3. 更新 line 的 rotation.x 和 rotation.y
4. 累加 morph 時間 t
5. 更新 morphTargetInfluences
6. 渲染場景
7. 更新 Stats 面板

**Called By**: WebGLRenderer animation loop  
**Frequency**: ~60 times per second (depends on refresh rate)

---

### onWindowResize()

視窗大小變更處理函式。

```typescript
function onWindowResize(): void
```

**Responsibilities**:
1. 更新相機 aspect ratio
2. 呼叫 camera.updateProjectionMatrix()
3. 更新渲染器尺寸

**Triggered By**: window 'resize' event

---

### generateMorphTargets(geometry)

產生 morph target 位置資料。

```typescript
function generateMorphTargets(geometry: THREE.BufferGeometry): void
```

**Parameters**:
| Name | Type | Description |
|------|------|-------------|
| geometry | BufferGeometry | 要附加 morph targets 的幾何物件 |

**Responsibilities**:
1. 建立與原始頂點數量相同的隨機位置陣列
2. 建立 Float32BufferAttribute
3. 設定 morphTarget.name
4. 將 morphTarget 加入 geometry.morphAttributes.position

**Side Effects**: 修改傳入的 geometry 物件

---

### showFallback()

顯示 WebGL 不支援時的替代內容。

```typescript
function showFallback(): void
```

**Responsibilities**:
1. 取得容器元素
2. 清空容器內容
3. 插入 fallback 圖片和錯誤訊息

---

## HTML Structure

### Required DOM Elements

```html
<div id="container"></div>
<div id="info">
    <a href="https://threejs.org">three.js</a> webgl - buffergeometry - lines
</div>
```

### Fallback Structure (動態生成)

```html
<div id="container">
    <div class="fallback">
        <img src="fallback.png" alt="WebGL BufferGeometry Lines 展示">
        <p>您的瀏覽器不支援 WebGL...</p>
    </div>
</div>
```

---

## CSS Requirements

### Base Styles

```css
body {
    margin: 0;
    background-color: #000;
    color: #fff;
    font-family: Monospace;
    overflow: hidden;
}

#container {
    width: 100%;
    height: 100vh;
}

#info {
    position: absolute;
    top: 10px;
    width: 100%;
    text-align: center;
    z-index: 100;
    pointer-events: none;
}

#info a {
    color: #0af;
    pointer-events: auto;
}
```

### Fallback Styles

```css
.fallback {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    text-align: center;
    padding: 20px;
}

.fallback img {
    max-width: 80%;
    max-height: 60vh;
    margin-bottom: 20px;
}

.fallback p {
    color: #999;
    max-width: 600px;
}
```

---

## Import Map Configuration

```json
{
    "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
    }
}
```

---

## External Dependencies

| Module | Import Path | Usage |
|--------|-------------|-------|
| THREE | three | 核心 3D 函式庫 |
| Stats | three/addons/libs/stats.module.js | FPS 統計面板 |

---

## Event Flow

```
Page Load
    │
    ▼
init()
    │
    ├──▶ WebGL OK ──▶ setAnimationLoop(animate)
    │                         │
    │                         ▼
    │                    animate() ◄─┐
    │                         │      │
    │                         └──────┘
    │
    └──▶ WebGL Fail ──▶ showFallback()

Window Resize
    │
    ▼
onWindowResize()
    │
    ▼
(continues animation loop)
```
