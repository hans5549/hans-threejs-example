# Data Model: WebGL Shader 動態視覺效果

**Feature**: 9-webgl-shader  
**Date**: 2025-12-01  
**Status**: Complete

## 概述

此功能為純視覺展示範例，不涉及持久化資料。資料模型主要描述執行時的狀態管理和 Three.js 物件結構。

---

## 執行時狀態

### 全域變數

| 變數名稱 | 類型 | 說明 |
|----------|------|------|
| `camera` | `THREE.OrthographicCamera` | 正交相機實例 |
| `scene` | `THREE.Scene` | 場景實例 |
| `renderer` | `THREE.WebGLRenderer` | WebGL 渲染器實例 |
| `uniforms` | `Object` | Shader uniform 參數物件 |

### Uniforms 結構

```javascript
uniforms = {
    time: {
        value: 1.0  // float - 動畫時間（秒）
    }
}
```

---

## Three.js 物件圖

```
Scene
└── Mesh
    ├── geometry: PlaneGeometry(2, 2)
    └── material: ShaderMaterial
        ├── uniforms: { time: { value: float } }
        ├── vertexShader: string (GLSL)
        └── fragmentShader: string (GLSL)

Camera: OrthographicCamera(-1, 1, 1, -1, 0, 1)

Renderer: WebGLRenderer
├── pixelRatio: devicePixelRatio
├── size: (innerWidth, innerHeight)
└── animationLoop: animate()
```

---

## Shader 資料流

### Vertex Shader

**輸入**:
| 名稱 | 類型 | 來源 | 說明 |
|------|------|------|------|
| `position` | `vec3` | Three.js 內建 | 頂點位置 |
| `uv` | `vec2` | Three.js 內建 | 紋理座標 [0, 1] |

**輸出**:
| 名稱 | 類型 | 說明 |
|------|------|------|
| `vUv` | `vec2` | 傳遞給 fragment shader 的 UV 座標 |
| `gl_Position` | `vec4` | 頂點裁切空間位置 |

### Fragment Shader

**輸入**:
| 名稱 | 類型 | 來源 | 說明 |
|------|------|------|------|
| `vUv` | `vec2` | Vertex Shader | UV 座標 [0, 1] |
| `time` | `float` | Uniform | 動畫時間（秒） |

**輸出**:
| 名稱 | 類型 | 說明 |
|------|------|------|
| `gl_FragColor` | `vec4` | 像素顏色 (RGBA) |

---

## 狀態更新流程

```
┌─────────────────────────────────────────────────────┐
│                   Animation Loop                     │
├─────────────────────────────────────────────────────┤
│                                                      │
│  1. 更新時間                                         │
│     uniforms.time.value = performance.now() / 1000  │
│                                                      │
│  2. 渲染場景                                         │
│     renderer.render(scene, camera)                  │
│                                                      │
│  3. GPU 執行 Shader                                  │
│     - Vertex Shader: 處理頂點位置和 UV              │
│     - Fragment Shader: 根據 UV 和 time 計算顏色     │
│                                                      │
│  4. 重複 (由 setAnimationLoop 自動呼叫)             │
│                                                      │
└─────────────────────────────────────────────────────┘
```

---

## 座標系統

### UV 座標空間

```
(0, 1) ─────────── (1, 1)
  │                  │
  │    PlaneGeometry │
  │       (2 x 2)    │
  │                  │
(0, 0) ─────────── (1, 0)
```

### Shader 內部座標 (p)

```glsl
vec2 p = -1.0 + 2.0 * vUv;
```

```
(-1, 1) ─────────── (1, 1)
   │                  │
   │  Normalized      │
   │  Coordinates     │
   │                  │
(-1, -1) ────────── (1, -1)
```

---

## 效能考量

| 項目 | 說明 |
|------|------|
| 記憶體使用 | 極低 - 單一平面幾何體 (4 頂點, 2 三角形) |
| GPU 運算 | 每像素執行一次 fragment shader |
| 狀態更新 | 每幀僅更新單一 uniform 值 |
| 無累積狀態 | Shader 無歷史依賴，可無限運行 |

---

## 驗證檢查點

- [ ] `uniforms.time.value` 持續遞增
- [ ] `gl_FragColor` 輸出有效的 RGBA 值
- [ ] 畫面持續動態變化
- [ ] 視窗調整後繼續正常渲染
