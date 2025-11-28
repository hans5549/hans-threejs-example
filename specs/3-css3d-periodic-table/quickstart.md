# Quickstart: CSS3D 週期表

**Feature**: 3-css3d-periodic-table  
**Date**: 2025-11-28

---

## 快速開始

### 1. 檔案位置

```
examples/css3d-periodic-table/
└── index.html    # 完整的單一 HTML 檔案
```

### 2. 執行方式

#### 方法 A: 本地伺服器 (推薦)

```bash
# 使用 Python
cd examples/css3d-periodic-table
python -m http.server 8080

# 使用 Node.js
npx serve .

# 使用 VS Code Live Server 擴充套件
# 右鍵 index.html → Open with Live Server
```

然後開啟 http://localhost:8080

#### 方法 B: 直接開啟

由於使用 CDN，現代瀏覽器可直接開啟 `index.html`（部分瀏覽器可能需要調整 CORS 設定）。

---

## 核心概念

### 技術架構

```
┌─────────────────────────────────────────────────────┐
│                    HTML Page                         │
├─────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   Three.js  │  │   TWEEN.js  │  │ CSS3DRender │ │
│  │   (Core)    │  │  (Animation)│  │   (DOM→3D)  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
├─────────────────────────────────────────────────────┤
│           TrackballControls (互動)                   │
├─────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────┐   │
│  │        118 × CSS3DObject (元素卡片)          │   │
│  └──────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────┤
│  targets.table | targets.sphere | targets.helix     │
│                | targets.grid                        │
└─────────────────────────────────────────────────────┘
```

### 核心流程

```javascript
// 1. 初始化
const camera = new THREE.PerspectiveCamera(...);
const scene = new THREE.Scene();
const renderer = new CSS3DRenderer();
const controls = new TrackballControls(camera, renderer.domElement);

// 2. 建立 118 個元素物件
for (let i = 0; i < table.length; i += 5) {
    const element = createElementCard(table, i);
    const object = new CSS3DObject(element);
    scene.add(object);
    objects.push(object);
}

// 3. 計算四種佈局的目標位置
calculateTableLayout(targets.table);
calculateSphereLayout(targets.sphere);
calculateHelixLayout(targets.helix);
calculateGridLayout(targets.grid);

// 4. 初始動畫：從隨機位置移動到週期表
transform(targets.table, 2000);

// 5. 動畫迴圈
function animate() {
    requestAnimationFrame(animate);
    TWEEN.update();
    controls.update();
}
```

---

## 關鍵程式碼

### 建立元素卡片

```javascript
function createElementCard(table, index) {
    const element = document.createElement('div');
    element.className = 'element';
    element.style.backgroundColor = `rgba(0,127,127,${Math.random() * 0.5 + 0.25})`;
    
    // 原子序
    const number = document.createElement('div');
    number.className = 'number';
    number.textContent = (index / 5) + 1;
    
    // 化學符號
    const symbol = document.createElement('div');
    symbol.className = 'symbol';
    symbol.textContent = table[index];
    
    // 詳細資訊
    const details = document.createElement('div');
    details.className = 'details';
    details.innerHTML = `${table[index + 1]}<br>${table[index + 2]}`;
    
    element.append(number, symbol, details);
    return element;
}
```

### 視圖切換動畫

```javascript
function transform(targets, duration) {
    TWEEN.removeAll();
    
    for (let i = 0; i < objects.length; i++) {
        const object = objects[i];
        const target = targets[i];
        
        // 位置動畫
        new TWEEN.Tween(object.position)
            .to({
                x: target.position.x,
                y: target.position.y,
                z: target.position.z
            }, Math.random() * duration + duration)
            .easing(TWEEN.Easing.Exponential.InOut)
            .start();
        
        // 旋轉動畫
        new TWEEN.Tween(object.rotation)
            .to({
                x: target.rotation.x,
                y: target.rotation.y,
                z: target.rotation.z
            }, Math.random() * duration + duration)
            .easing(TWEEN.Easing.Exponential.InOut)
            .start();
    }
    
    // 觸發渲染更新
    new TWEEN.Tween({})
        .to({}, duration * 2)
        .onUpdate(render)
        .start();
}
```

---

## 使用者操作

| 操作 | 效果 |
|------|------|
| 滑鼠左鍵拖曳 | 旋轉視角 |
| 滑鼠滾輪 | 縮放視角 |
| 滑鼠右鍵拖曳 | 平移視角 |
| 點擊 TABLE | 切換至週期表視圖 |
| 點擊 SPHERE | 切換至球形視圖 |
| 點擊 HELIX | 切換至螺旋視圖 |
| 點擊 GRID | 切換至網格視圖 |
| 滑鼠移至卡片上 | 卡片發光效果增強 |

---

## 依賴項

| 套件 | 版本 | 用途 |
|------|------|------|
| three | 0.160.0 | 3D 核心函式庫 |
| three/addons/libs/tween.module.js | - | 補間動畫 |
| three/addons/controls/TrackballControls.js | - | 相機控制 |
| three/addons/renderers/CSS3DRenderer.js | - | CSS 3D 渲染 |

所有依賴透過 unpkg CDN 載入，無需本地安裝。

---

## 延伸學習

- [Three.js CSS3DRenderer 文件](https://threejs.org/docs/#examples/en/renderers/CSS3DRenderer)
- [TWEEN.js 文件](https://github.com/tweenjs/tween.js)
- [TrackballControls 文件](https://threejs.org/docs/#examples/en/controls/TrackballControls)
