# Quickstart Guide: WebGL Effects ASCII

**Feature**: 017-effects-ascii  
**Date**: 2025-01-27

## Overview

本指南說明如何快速建立和運行 WebGL Effects ASCII 範例。

---

## Prerequisites

- 現代瀏覽器（Chrome, Firefox, Safari, Edge）支援 WebGL
- 本地 HTTP 伺服器（用於測試 ES6 modules）

---

## Quick Setup

### Step 1: 建立目錄結構

```bash
mkdir -p examples/webgl-effects-ascii
```

### Step 2: 建立 index.html

建立 `examples/webgl-effects-ascii/index.html` 檔案，包含以下結構：

```html
<!DOCTYPE html>
<html>
<head>
    <title>three.js webgl - effects - ascii</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, user-scalable=no, minimum-scale=1.0, maximum-scale=1.0">
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background: #000000;
        }
    </style>
</head>
<body>
    <script type="importmap">
    {
        "imports": {
            "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
            "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
        }
    }
    </script>
    <script type="module">
        // 應用程式碼在此
    </script>
</body>
</html>
```

### Step 3: 添加 JavaScript 程式碼

在 `<script type="module">` 區塊中添加：

```javascript
import * as THREE from 'three';
import { AsciiEffect } from 'three/addons/effects/AsciiEffect.js';
import { TrackballControls } from 'three/addons/controls/TrackballControls.js';

let camera, scene, renderer, effect, controls;
let sphere;
const start = Date.now();

init();

function init() {
    // 1. 建立相機
    camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 1, 1000);
    camera.position.y = 150;
    camera.position.z = 500;

    // 2. 建立場景
    scene = new THREE.Scene();
    scene.background = new THREE.Color(0x000000);

    // 3. 建立光源
    const light1 = new THREE.PointLight(0xffffff, 1.5, 0);
    light1.position.set(500, 500, 500);
    scene.add(light1);

    const light2 = new THREE.PointLight(0xffffff, 0.25, 0);
    light2.position.set(-500, -500, -500);
    scene.add(light2);

    // 4. 建立球體
    sphere = new THREE.Mesh(
        new THREE.SphereGeometry(200, 20, 10),
        new THREE.MeshPhongMaterial({ flatShading: true })
    );
    scene.add(sphere);

    // 5. 建立平面
    const plane = new THREE.Mesh(
        new THREE.PlaneGeometry(400, 400),
        new THREE.MeshPhongMaterial({ flatShading: true })
    );
    plane.position.y = -200;
    plane.rotation.x = -Math.PI / 2;
    scene.add(plane);

    // 6. 建立渲染器
    renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);

    // 7. 建立 ASCII 效果
    effect = new AsciiEffect(renderer, ' .:-+*=%@#', { invert: true });
    effect.setSize(window.innerWidth, window.innerHeight);
    effect.domElement.style.color = 'white';
    effect.domElement.style.backgroundColor = 'black';
    document.body.appendChild(effect.domElement);

    // 8. 建立控制器
    controls = new TrackballControls(camera, effect.domElement);

    // 9. 監聽視窗大小改變
    window.addEventListener('resize', onWindowResize);

    // 10. 開始動畫
    animate();
}

function animate() {
    requestAnimationFrame(animate);

    const timer = Date.now() - start;

    // 球體彈跳
    sphere.position.y = Math.abs(Math.sin(timer * 0.002)) * 150;

    // 球體旋轉
    sphere.rotation.x = timer * 0.0003;
    sphere.rotation.z = timer * 0.0002;

    controls.update();
    effect.render(scene, camera);
}

function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();

    renderer.setSize(window.innerWidth, window.innerHeight);
    effect.setSize(window.innerWidth, window.innerHeight);
}
```

---

## Running the Example

### Option 1: VS Code Live Server

1. 安裝 Live Server 擴充功能
2. 右鍵點擊 `index.html` → "Open with Live Server"

### Option 2: Python HTTP Server

```bash
cd examples/webgl-effects-ascii
python -m http.server 8080
# 開啟 http://localhost:8080
```

### Option 3: Node.js http-server

```bash
npx http-server examples/webgl-effects-ascii -p 8080
# 開啟 http://localhost:8080
```

---

## Expected Result

開啟頁面後，您應該看到：

1. ✅ 黑色背景上的白色 ASCII 字元
2. ✅ 一個持續上下彈跳的球體
3. ✅ 一個靜止的地板平面
4. ✅ 球體持續旋轉

### 互動測試

| 動作 | 預期結果 |
|------|----------|
| 左鍵拖曳 | 相機繞場景旋轉 |
| 滾輪滾動 | 放大/縮小視角 |
| 右鍵拖曳 | 相機平移 |
| 調整視窗大小 | ASCII 渲染自動適應 |

---

## Troubleshooting

### 問題：畫面空白

1. 開啟瀏覽器開發者工具（F12）
2. 檢查 Console 是否有錯誤訊息
3. 確認使用 HTTP 伺服器而非直接開啟檔案

### 問題：CORS 錯誤

確保透過 HTTP 伺服器訪問，而非 `file://` 協定。

### 問題：模組載入失敗

檢查 importmap 配置是否正確，CDN URL 是否可訪問。

---

## File Structure

```
examples/
└── webgl-effects-ascii/
    └── index.html      # 單一檔案包含所有程式碼
```

---

## Next Steps

- 嘗試修改 `charSet` 參數使用不同字元
- 調整 `invert` 選項查看黑字白底效果
- 修改球體動畫參數調整彈跳速度和高度
