# WebGL Shader 快速入門

**Feature**: 9-webgl-shader  
**Date**: 2025-12-01

## 功能概述

使用 Three.js 實作的 WebGL Shader 範例，展示經典的 Monjori 程式化視覺藝術效果。這是一個全螢幕的即時渲染動畫，透過自訂 GLSL shader 產生動態的色彩圖案。

## 相關連結

- [Three.js 官方範例](https://threejs.org/examples/#webgl_shader)
- [原始碼參考](https://github.com/mrdoob/three.js/blob/master/examples/webgl_shader.html)
- [Monjori 原作 (Pouët)](http://www.pouet.net/prod.php?which=52761)

## 快速開始

### 1. 開啟範例

在瀏覽器中開啟 `examples/webgl-shader/index.html`

### 2. 預期效果

- 全螢幕顯示動態的色彩視覺效果
- 色彩會隨時間持續變化和流動
- 調整視窗大小時效果會自動適應

## 技術亮點

| 技術 | 說明 |
|------|------|
| ShaderMaterial | 使用自訂 GLSL shader 的 Three.js 材質 |
| OrthographicCamera | 正交相機用於 2D 全螢幕渲染 |
| Fragment Shader | Monjori 演算法產生程式化視覺藝術 |
| Uniform | 透過 uniform 變數傳遞時間參數給 shader |

## 核心程式碼結構

```javascript
// 1. 建立正交相機 (覆蓋 [-1, 1] 的標準化空間)
camera = new THREE.OrthographicCamera(-1, 1, 1, -1, 0, 1);

// 2. 建立覆蓋整個視窗的平面
geometry = new THREE.PlaneGeometry(2, 2);

// 3. 建立 shader 材質
uniforms = { time: { value: 1.0 } };
material = new THREE.ShaderMaterial({
    uniforms: uniforms,
    vertexShader: /* GLSL */,
    fragmentShader: /* GLSL (Monjori) */
});

// 4. 動畫迴圈中更新時間
function animate() {
    uniforms.time.value = performance.now() / 1000;
    renderer.render(scene, camera);
}
```

## 系統需求

- 支援 WebGL 1.0+ 的現代瀏覽器
- 建議: Chrome, Firefox, Safari, Edge 最新版本

## 檔案結構

```
examples/webgl-shader/
└── index.html      # 主要展示頁面 (含所有程式碼)
```

## 延伸學習

- [The Book of Shaders](https://thebookofshaders.com/) - GLSL 程式設計入門
- [Shadertoy](https://www.shadertoy.com/) - 線上 shader 編輯器和社群
- [Three.js ShaderMaterial 文件](https://threejs.org/docs/#api/en/materials/ShaderMaterial)
