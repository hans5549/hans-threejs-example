# Shader Uniforms Contract

**Feature**: 14-particle-earth  
**Date**: 2025-12-01  
**Type**: GLSL Shader Interface

## Vertex Shader Uniforms

| Name | Type | Description | Source |
|------|------|-------------|--------|
| `map` | `sampler2D` | 地球紋理貼圖 | TextureLoader |
| `pointSize` | `float` | 粒子大小 (像素) | GUI / 預設值 |
| `time` | `float` | 動畫時間 (秒) | animate() 累加 |
| `rotationSpeed` | `float` | Y 軸旋轉速度 | GUI / 預設值 |

## Vertex Shader Attributes

| Name | Type | Description | Size |
|------|------|-------------|------|
| `position` | `vec3` | 粒子球面座標位置 | numParticles * 3 |

## Vertex Shader Varyings (輸出到 Fragment)

| Name | Type | Description |
|------|------|-------------|
| `vUv` | `vec2` | 紋理 UV 座標 |
| `vColor` | `vec3` | 從紋理取樣的顏色 |

## Fragment Shader Uniforms

| Name | Type | Description |
|------|------|-------------|
| `map` | `sampler2D` | 地球紋理貼圖 |
| `opacity` | `float` | 全域透明度 |

## Shader Code Interface

### Vertex Shader

```glsl
// Uniforms
uniform sampler2D map;
uniform float pointSize;
uniform float time;
uniform float rotationSpeed;

// Attributes (from BufferGeometry)
// position - vec3, provided by Three.js

// Varyings
varying vec2 vUv;
varying vec3 vColor;

// Constants
#define PI 3.14159265359

void main() {
    // 1. 套用 Y 軸旋轉
    float angle = time * rotationSpeed;
    mat3 rotationMatrix = mat3(
        cos(angle), 0.0, sin(angle),
        0.0,        1.0, 0.0,
        -sin(angle), 0.0, cos(angle)
    );
    vec3 rotatedPosition = rotationMatrix * position;
    
    // 2. 計算 UV 座標 (等距柱狀投影)
    vec3 normalized = normalize(rotatedPosition);
    float u = 0.5 + atan(normalized.z, normalized.x) / (2.0 * PI);
    float v = 0.5 - asin(normalized.y) / PI;
    vUv = vec2(u, v);
    
    // 3. 從紋理取樣顏色
    vColor = texture2D(map, vUv).rgb;
    
    // 4. 設定粒子大小
    gl_PointSize = pointSize;
    
    // 5. 計算最終位置
    gl_Position = projectionMatrix * modelViewMatrix * vec4(rotatedPosition, 1.0);
}
```

### Fragment Shader

```glsl
// Uniforms
uniform float opacity;

// Varyings
varying vec2 vUv;
varying vec3 vColor;

void main() {
    // 使用從 vertex shader 傳遞的顏色
    gl_FragColor = vec4(vColor, opacity);
}
```

## JavaScript Uniforms Object

```javascript
const uniforms = {
    map: { value: earthTexture },
    pointSize: { value: 2.0 },
    time: { value: 0.0 },
    rotationSpeed: { value: 0.001 },
    opacity: { value: 0.8 }
};
```

## GUI Bindings

| Uniform | GUI Control | Min | Max | Step |
|---------|-------------|-----|-----|------|
| pointSize | Slider | 1.0 | 10.0 | 0.1 |
| rotationSpeed | Slider | 0.0 | 0.01 | 0.0001 |
| opacity | Slider | 0.0 | 1.0 | 0.01 |

## Update Contract

在 `animate()` 函式中必須更新：

```javascript
// 每幀更新 time uniform
uniforms.time.value += deltaTime;

// 或使用絕對時間
uniforms.time.value = performance.now() * 0.001;
```
