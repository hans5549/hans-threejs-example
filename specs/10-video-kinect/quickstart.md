# Quickstart: WebGL Video Kinect 深度影片點雲視覺化

**Feature**: 10-video-kinect  
**Date**: 2025-12-01  
**Status**: Ready for Implementation

## 概述

本指南提供快速上手 WebGL Video Kinect 範例的步驟，幫助開發者理解並開始實作。

---

## 快速開始

### 1. 建立檔案結構

```
examples/
└── webgl-video-kinect/
    ├── index.html
    └── textures/
        ├── kinect.webm
        └── kinect.mp4
```

### 2. 取得 Kinect 影片素材

從 Three.js 官方範例下載：
- https://threejs.org/examples/textures/kinect.webm
- https://threejs.org/examples/textures/kinect.mp4

### 3. 基本 HTML 模板

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, user-scalable=no, minimum-scale=1.0, maximum-scale=1.0">
  <title>Three.js WebGL Video Kinect</title>
  <style>
    body {
      margin: 0;
      background-color: #000000;
      overflow: hidden;
    }
    #info {
      position: absolute;
      top: 10px;
      width: 100%;
      text-align: center;
      color: #fff;
      font-family: Arial, sans-serif;
    }
    a { color: #9cf; }
  </style>
</head>
<body>
  <div id="info">
    <a href="https://threejs.org">three.js</a> webgl - kinect depth video
  </div>
  
  <video id="video" loop muted crossOrigin="anonymous" playsinline style="display:none">
    <source src="textures/kinect.webm" type="video/webm">
    <source src="textures/kinect.mp4" type="video/mp4">
  </video>

  <script type="importmap">
  {
    "imports": {
      "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
      "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
    }
  }
  </script>

  <script type="module">
    // 實作程式碼放這裡
  </script>
</body>
</html>
```

---

## 核心實作步驟

### Step 1: 匯入相依套件

```javascript
import * as THREE from 'three';
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';
```

### Step 2: 定義全域變數

```javascript
let camera, scene, renderer;
let mouse = { x: 0, y: 0 };
let center = new THREE.Vector3(0, 0, -1000);
let windowHalfX = window.innerWidth / 2;
let windowHalfY = window.innerHeight / 2;
```

### Step 3: 定義 Shaders

**Vertex Shader:**
```javascript
const vertexShader = `
  uniform sampler2D map;
  uniform float width;
  uniform float height;
  uniform float nearClipping, farClipping;
  uniform float pointSize;
  uniform float zOffset;
  
  varying vec2 vUv;
  
  const float XtoZ = 1.11146;
  const float YtoZ = 0.83359;
  
  void main() {
    vUv = vec2(position.x / width, position.y / height);
    vec4 color = texture2D(map, vUv);
    float depth = (color.r + color.g + color.b) / 3.0;
    float z = (1.0 - depth) * (farClipping - nearClipping) + nearClipping;
    
    vec4 pos = vec4(
      (position.x / width - 0.5) * z * XtoZ,
      (position.y / height - 0.5) * z * YtoZ,
      -z + zOffset,
      1.0
    );
    
    gl_PointSize = pointSize;
    gl_Position = projectionMatrix * modelViewMatrix * pos;
  }
`;
```

**Fragment Shader:**
```javascript
const fragmentShader = `
  uniform sampler2D map;
  varying vec2 vUv;
  
  void main() {
    vec4 color = texture2D(map, vUv);
    gl_FragColor = vec4(color.r, color.g, color.b, 0.2);
  }
`;
```

### Step 4: 建立 VideoTexture

```javascript
const video = document.getElementById('video');
const texture = new THREE.VideoTexture(video);
texture.minFilter = THREE.NearestFilter;
texture.generateMipmaps = false;
```

### Step 5: 建立點雲幾何

```javascript
const width = 640, height = 480;
const geometry = new THREE.BufferGeometry();
const positions = new Float32Array(width * height * 3);

for (let i = 0, j = 0; i < positions.length; i += 3, j++) {
  positions[i] = j % width;
  positions[i + 1] = Math.floor(j / width);
}

geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
```

### Step 6: 建立 ShaderMaterial

```javascript
const uniforms = {
  map: { value: texture },
  width: { value: width },
  height: { value: height },
  nearClipping: { value: 850 },
  farClipping: { value: 4000 },
  pointSize: { value: 2 },
  zOffset: { value: 1000 }
};

const material = new THREE.ShaderMaterial({
  uniforms,
  vertexShader,
  fragmentShader,
  blending: THREE.AdditiveBlending,
  depthTest: false,
  depthWrite: false,
  transparent: true
});
```

### Step 7: 建立場景

```javascript
scene = new THREE.Scene();
center = new THREE.Vector3(0, 0, -1000);

camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 1, 10000);
camera.position.set(0, 0, 500);

const mesh = new THREE.Points(geometry, material);
scene.add(mesh);
```

### Step 8: 建立渲染器和 GUI

```javascript
renderer = new THREE.WebGLRenderer();
renderer.setPixelRatio(window.devicePixelRatio);
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

const gui = new GUI();
gui.add(uniforms.nearClipping, 'value', 1, 10000, 1).name('Near Clipping');
gui.add(uniforms.farClipping, 'value', 1, 10000, 1).name('Far Clipping');
gui.add(uniforms.pointSize, 'value', 1, 10, 1).name('Point Size');
gui.add(uniforms.zOffset, 'value', 0, 4000, 1).name('Z Offset');
```

### Step 9: 設定事件監聽

```javascript
document.addEventListener('mousemove', (event) => {
  mouse.x = (event.clientX - windowHalfX) * 8;
  mouse.y = (event.clientY - windowHalfY) * 8;
});

window.addEventListener('resize', () => {
  windowHalfX = window.innerWidth / 2;
  windowHalfY = window.innerHeight / 2;
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
```

### Step 10: 動畫迴圈

```javascript
function animate() {
  requestAnimationFrame(animate);
  
  camera.position.x += (mouse.x - camera.position.x) * 0.05;
  camera.position.y += (-mouse.y - camera.position.y) * 0.05;
  camera.lookAt(center);
  
  renderer.render(scene, camera);
}

video.play();
animate();
```

---

## 驗證清單

- [ ] 點雲正確顯示
- [ ] 滑鼠移動控制相機
- [ ] GUI 滑桿可調整參數
- [ ] 視窗調整正確響應
- [ ] 效能達到 30+ FPS

---

## 常見問題

### Q: 影片無法播放？
A: 確保影片有正確的 MIME 類型，且瀏覽器支援 WebM/MP4。使用 `muted` 屬性允許自動播放。

### Q: 點雲不顯示？
A: 檢查 `nearClipping` 和 `farClipping` 的值是否正確，確保深度值在範圍內。

### Q: 效能問題？
A: 確認 `pointSize` 不要設太大，GPU 要處理 307,200 個點。

---

## 下一步

完成基本實作後，可以嘗試：
1. 添加色彩映射
2. 實作點雲旋轉
3. 加入播放/暫停控制
4. 支援其他深度影片格式
