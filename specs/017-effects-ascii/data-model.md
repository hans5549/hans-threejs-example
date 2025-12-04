# Data Model: WebGL Effects ASCII

**Feature**: 017-effects-ascii  
**Date**: 2025-01-27  
**Status**: Complete

## Overview

本文件定義 WebGL Effects ASCII 範例中的資料模型和實體關係。

---

## Entities

### 1. Scene Configuration

場景的整體配置設定。

```typescript
interface SceneConfig {
    background: number;        // 背景顏色 (0x000000 黑色)
}
```

### 2. Camera Configuration

相機的初始配置。

```typescript
interface CameraConfig {
    fov: number;              // 視野角度 (70)
    aspect: number;           // 長寬比 (window.innerWidth / window.innerHeight)
    near: number;             // 近裁剪面 (1)
    far: number;              // 遠裁剪面 (1000)
    position: {
        x: number;            // 0
        y: number;            // 150
        z: number;            // 500
    };
}
```

### 3. Sphere Entity

主要動畫物件 - 彈跳球體。

```typescript
interface SphereEntity {
    geometry: {
        radius: number;       // 200
        widthSegments: number;  // 20
        heightSegments: number; // 10
    };
    material: {
        type: 'MeshPhongMaterial';
        flatShading: boolean; // true
    };
    animation: {
        bounceFrequency: number;  // 2 (Math.sin 的係數)
        bounceAmplitude: number;  // 150 (彈跳高度)
        rotationSpeedX: number;   // 0.0003
        rotationSpeedZ: number;   // 0.0002
    };
}
```

### 4. Plane Entity

地板物件。

```typescript
interface PlaneEntity {
    geometry: {
        width: number;        // 400
        height: number;       // 400
    };
    material: {
        type: 'MeshPhongMaterial';
        flatShading: boolean; // true
    };
    position: {
        y: number;            // -200
    };
    rotation: {
        x: number;            // -Math.PI / 2 (水平放置)
    };
}
```

### 5. Light Configuration

光源設定。

```typescript
interface LightConfig {
    lights: Array<{
        type: 'PointLight';
        color: number;
        intensity: number;
        distance: number;
        position: {
            x: number;
            y: number;
            z: number;
        };
    }>;
}

// 實際配置
const lightConfig: LightConfig = {
    lights: [
        {
            type: 'PointLight',
            color: 0xffffff,
            intensity: 1.5,
            distance: 0,
            position: { x: 500, y: 500, z: 500 }
        },
        {
            type: 'PointLight',
            color: 0xffffff,
            intensity: 0.25,
            distance: 0,
            position: { x: -500, y: -500, z: -500 }
        }
    ]
};
```

### 6. ASCII Effect Configuration

AsciiEffect 的配置。

```typescript
interface AsciiEffectConfig {
    charSet: string;          // ' .:-+*=%@#'
    options: {
        invert: boolean;      // true (白字黑底)
        resolution?: number;  // 預設 0.15
        scale?: number;       // 預設 1
        color?: boolean;      // 預設 false
    };
    style: {
        color: string;        // 'white'
        backgroundColor: string; // 'black'
    };
}
```

### 7. Controls Configuration

TrackballControls 的配置。

```typescript
interface ControlsConfig {
    type: 'TrackballControls';
    // 使用預設值
}
```

---

## Entity Relationships

```
┌─────────────────────────────────────────────────────────┐
│                         Scene                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   Sphere    │  │    Plane    │  │   Lights    │     │
│  │  (animated) │  │  (static)   │  │  (2 point)  │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    WebGLRenderer                         │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                     AsciiEffect                          │
│  - Converts WebGL output to ASCII characters            │
│  - domElement → appended to document.body               │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                  TrackballControls                       │
│  - Listens to effect.domElement                         │
│  - Controls camera rotation/zoom/pan                    │
└─────────────────────────────────────────────────────────┘
```

---

## State Transitions

### Animation State

```
[Initial] ──► [Running] ◄──────────────────────┐
                 │                              │
                 ▼                              │
         [Update Position]                      │
                 │                              │
                 ▼                              │
         [Update Rotation]                      │
                 │                              │
                 ▼                              │
           [Render Frame] ─────────────────────►┘
```

### Resize Event State

```
[Window Resize Event]
         │
         ▼
[Update Camera Aspect]
         │
         ▼
[Update Camera Projection]
         │
         ▼
[Update Renderer Size]
         │
         ▼
[Update Effect Size]
         │
         ▼
[Continue Animation Loop]
```

---

## Validation Rules

### Camera Validation
- `fov` 必須在 1-179 度之間
- `aspect` 必須為正數
- `near` 必須小於 `far`

### Sphere Validation
- `radius` 必須為正數
- `widthSegments` 和 `heightSegments` 必須為正整數

### ASCII Effect Validation
- `charSet` 不可為空字串
- 字元應從暗到亮排列（使用 `invert: true` 時會反轉）

---

## Constants

```javascript
// 預設值常數
const DEFAULTS = {
    CAMERA_FOV: 70,
    CAMERA_NEAR: 1,
    CAMERA_FAR: 1000,
    CAMERA_Y: 150,
    CAMERA_Z: 500,
    
    SPHERE_RADIUS: 200,
    SPHERE_WIDTH_SEGMENTS: 20,
    SPHERE_HEIGHT_SEGMENTS: 10,
    
    PLANE_SIZE: 400,
    PLANE_Y: -200,
    
    BOUNCE_AMPLITUDE: 150,
    BOUNCE_FREQUENCY: 2,
    ROTATION_SPEED_X: 0.0003,
    ROTATION_SPEED_Z: 0.0002,
    
    ASCII_CHARSET: ' .:-+*=%@#',
    BACKGROUND_COLOR: 0x000000
};
```
