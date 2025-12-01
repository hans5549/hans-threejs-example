# Data Model: WebGL Custom Attributes Lines

**Feature**: 12-custom-attributes-lines  
**Date**: 2025-12-01  
**Status**: Complete

## 核心實體

### 1. Scene Configuration

場景配置物件，定義 3D 環境的基本設定。

| 屬性 | 類型 | 描述 | 預設值 |
|------|------|------|--------|
| backgroundColor | Color | 場景背景色 | 0x050505 |
| cameraFov | number | 相機視野角度 | 30 |
| cameraNear | number | 相機近裁剪面 | 1 |
| cameraFar | number | 相機遠裁剪面 | 10000 |
| cameraZ | number | 相機 Z 軸位置 | 400 |

### 2. Uniforms Object

傳遞給 ShaderMaterial 的 uniform 變數集合。

| 屬性 | 類型 | 描述 | 初始值 | 動態更新 |
|------|------|------|--------|----------|
| amplitude | { value: number } | 位移振幅係數 | 5.0 | ✅ 每幀更新 (sin wave) |
| opacity | { value: number } | 線條透明度 | 0.3 | ❌ 固定 |
| color | { value: Color } | 基礎顏色 | 0xffffff | ✅ 每幀更新 (HSL offset) |

### 3. Buffer Attributes

附加到 geometry 的自訂頂點屬性。

#### displacement Attribute
| 屬性 | 類型 | 描述 |
|------|------|------|
| data | Float32Array | 每個頂點的位移向量 (x, y, z) |
| itemSize | number | 3 (vec3) |
| count | number | 等於 geometry.attributes.position.count |
| needsUpdate | boolean | 每幀設為 true 以同步 GPU |

#### customColor Attribute
| 屬性 | 類型 | 描述 |
|------|------|------|
| data | Float32Array | 每個頂點的 RGB 顏色 |
| itemSize | number | 3 (vec3) |
| count | number | 等於 geometry.attributes.position.count |
| needsUpdate | boolean | 初始化後不需更新 |

### 4. TextGeometry Parameters

TextGeometry 建立參數。

| 參數 | 類型 | 描述 | 值 |
|------|------|------|-----|
| text | string | 顯示文字 | "three.js" |
| font | Font | 載入的字型物件 | Helvetiker Bold |
| size | number | 文字大小 | 50 |
| depth | number | 文字深度 (Z 軸) | 15 |
| curveSegments | number | 曲線分段數 | 10 |
| bevelEnabled | boolean | 啟用斜角 | true |
| bevelThickness | number | 斜角厚度 | 5 |
| bevelSize | number | 斜角大小 | 1.5 |
| bevelSegments | number | 斜角分段數 | 10 |

### 5. ShaderMaterial Configuration

ShaderMaterial 建立配置。

| 屬性 | 類型 | 描述 | 值 |
|------|------|------|-----|
| uniforms | object | Uniform 變數物件 | 參見 Uniforms Object |
| vertexShader | string | Vertex shader GLSL 程式碼 | 參見 Shader 定義 |
| fragmentShader | string | Fragment shader GLSL 程式碼 | 參見 Shader 定義 |
| blending | BlendingMode | 混合模式 | THREE.AdditiveBlending |
| depthTest | boolean | 深度測試 | false |
| transparent | boolean | 透明模式 | true |

---

## Shader 定義

### Vertex Shader

```glsl
uniform float amplitude;

attribute vec3 displacement;
attribute vec3 customColor;

varying vec3 vColor;

void main() {
    // 應用位移
    vec3 newPosition = position + amplitude * displacement;
    
    // 傳遞顏色到 fragment shader
    vColor = customColor;
    
    // 計算最終位置
    gl_Position = projectionMatrix * modelViewMatrix * vec4(newPosition, 1.0);
}
```

### Fragment Shader

```glsl
uniform vec3 color;
uniform float opacity;

varying vec3 vColor;

void main() {
    // 混合頂點顏色和 uniform 顏色
    gl_FragColor = vec4(vColor * color, opacity);
}
```

---

## 資料流程

```
┌─────────────────────────────────────────────────────────────────┐
│                        Initialization                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FontLoader.load() ──► TextGeometry ──► BufferAttributes        │
│        │                    │               │                    │
│        ▼                    ▼               ▼                    │
│    (on error)          geometry      displacement + customColor  │
│        │                    │               │                    │
│        ▼                    └───────┬───────┘                    │
│   BoxGeometry                       │                            │
│        │                            ▼                            │
│        └────────────────► THREE.Line(geometry, shaderMaterial)   │
│                                     │                            │
│                                     ▼                            │
│                               scene.add(line)                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        Animation Loop                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Date.now() ──► time                                            │
│        │                                                         │
│        ├──► line.rotation.y = 0.25 * time                       │
│        │                                                         │
│        ├──► uniforms.amplitude.value = Math.sin(0.5 * time)     │
│        │                                                         │
│        ├──► uniforms.color.value.offsetHSL(0.0005, 0, 0)        │
│        │                                                         │
│        └──► displacement.array[i] += random offset              │
│                    │                                             │
│                    ▼                                             │
│             displacement.needsUpdate = true                      │
│                    │                                             │
│                    ▼                                             │
│             renderer.render(scene, camera)                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 狀態轉換

### Line 物件生命週期

```
[Uninitialized] ──FontLoader.load()──► [Loading]
                                          │
                    ┌─────────────────────┴─────────────────────┐
                    │                                           │
                    ▼ (success)                                 ▼ (error)
            [TextGeometry Created]                    [BoxGeometry Created]
                    │                                           │
                    └─────────────────────┬─────────────────────┘
                                          │
                                          ▼
                              [Attributes Attached]
                                          │
                                          ▼
                              [Line Added to Scene]
                                          │
                                          ▼
                                   [Animating]
                                          │
                           ┌──────────────┴──────────────┐
                           │                              │
                           ▼                              ▼
                   [Rotating]                    [Displacement Updating]
                           │                              │
                           └──────────────┬──────────────┘
                                          │
                                          ▼
                                   [Rendering]
                                          │
                                          └───────────────► [Animating] (loop)
```

---

## 驗證規則

### Uniforms 驗證
- `amplitude.value` 應為有限數值
- `opacity.value` 應在 [0, 1] 範圍內
- `color.value` 應為有效的 THREE.Color 物件

### Attributes 驗證
- `displacement.count` 必須等於 `geometry.attributes.position.count`
- `customColor.count` 必須等於 `geometry.attributes.position.count`
- 所有陣列值應為有限數值（非 NaN、非 Infinity）

### Geometry 驗證
- TextGeometry 或 BoxGeometry 必須成功建立
- geometry 必須有 `position` attribute
