# Data Model: WebGPU Custom Lighting Model

**Feature**: 13-webgpu-lights-custom  
**Date**: 2025-12-01  
**Source**: [spec.md](./spec.md)

## Entity Overview

本範例為前端靜態頁面，無持久化資料儲存。以下描述運行時的主要物件結構。

```
┌─────────────────────────────────────────────────────────────┐
│                         Scene                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   Point Cloud                        │   │
│  │  - 500,000 vertices                                  │   │
│  │  - PointsNodeMaterial                                │   │
│  │  - CustomLightingModel                               │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │ Light 1  │  │ Light 2  │  │ Light 3  │                  │
│  │ (Orange) │  │ (Blue)   │  │ (Green)  │                  │
│  │ 0xffaa00 │  │ 0x0040ff │  │ 0x80ff80 │                  │
│  │  + Mesh  │  │  + Mesh  │  │  + Mesh  │                  │
│  └──────────┘  └──────────┘  └──────────┘                  │
└─────────────────────────────────────────────────────────────┘
              │
              ▼
    ┌─────────────────┐
    │ WebGPURenderer  │
    │ + OrbitControls │
    │ + Camera        │
    └─────────────────┘
```

## Core Entities

### 1. Scene Configuration

| Property | Type | Value | Description |
|----------|------|-------|-------------|
| background | Color | 0x000000 | 純黑色背景 |

### 2. Camera

| Property | Type | Value | Description |
|----------|------|-------|-------------|
| type | - | PerspectiveCamera | 透視相機 |
| fov | number | 70 | 視野角度 |
| near | number | 0.1 | 近裁切面 |
| far | number | 10 | 遠裁切面 |
| position.z | number | 1.5 | 初始 Z 位置 |

### 3. PointLight (x3)

| Property | Type | Light 1 | Light 2 | Light 3 | Description |
|----------|------|---------|---------|---------|-------------|
| color | hex | 0xffaa00 | 0x0040ff | 0x80ff80 | 橙/藍/綠 |
| intensity | number | 0.1 | 0.1 | 0.1 | 光源強度 |
| distance | number | 1 | 1 | 1 | 有效距離 |

### 4. Light Indicator Sphere

| Property | Type | Value | Description |
|----------|------|-------|-------------|
| geometry | SphereGeometry | radius: 0.02, segments: 16x8 | 小球幾何 |
| material | NodeMaterial | - | 自發光材質 |
| material.colorNode | color() | 同光源色 | 純色節點 |
| material.lightsNode | lights() | [] | 空陣列忽略光源 |

### 5. Point Cloud

| Property | Type | Value | Description |
|----------|------|-------|-------------|
| pointCount | number | 500,000 | 粒子數量 |
| distribution | - | Random uniform | 均勻隨機分佈 |
| range | Vector3 | ±1.5 (3x3x3 cube) | 空間範圍 |
| geometry | BufferGeometry | setFromPoints() | 頂點幾何 |
| material | PointsNodeMaterial | - | 點雲材質 |
| material.lightsNode | - | lightingModelContext | 自訂光照 |

### 6. CustomLightingModel

```
CustomLightingModel extends THREE.LightingModel
├── direct({ lightColor, reflectedLight })
│   └── reflectedLight.directDiffuse.addAssign(lightColor)
```

| Method | Parameters | Return | Description |
|--------|------------|--------|-------------|
| direct | { lightColor, reflectedLight } | void | 直接光照計算 |

### 7. Animation State

| Property | Type | Update | Description |
|----------|------|--------|-------------|
| time | number | Date.now() * 0.001 | 動畫時間（秒） |
| scale | number | 0.5 | 光源移動幅度 |
| scene.rotation.y | number | time * 0.1 | 場景旋轉 |

### Light Animation Formulas

| Light | X | Y | Z |
|-------|---|---|---|
| Light 1 | sin(time * 0.7) * scale | cos(time * 0.5) * scale | cos(time * 0.3) * scale |
| Light 2 | cos(time * 0.3) * scale | sin(time * 0.5) * scale | sin(time * 0.7) * scale |
| Light 3 | sin(time * 0.7) * scale | cos(time * 0.3) * scale | sin(time * 0.5) * scale |

## OrbitControls Configuration

| Property | Type | Value | Description |
|----------|------|-------|-------------|
| minDistance | number | 0 | 最小縮放距離 |
| maxDistance | number | 4 | 最大縮放距離 |

## State Diagram

```
                    ┌─────────────────┐
                    │    Page Load    │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Check WebGPU   │
                    │   Support       │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
    ┌─────────────────┐           ┌─────────────────┐
    │    Supported    │           │  Not Supported  │
    └────────┬────────┘           └────────┬────────┘
             │                             │
             ▼                             ▼
    ┌─────────────────┐           ┌─────────────────┐
    │  Initialize     │           │  Show Fallback  │
    │  Scene & Render │           │  Image + Text   │
    └────────┬────────┘           └─────────────────┘
             │
             ▼
    ┌─────────────────┐
    │ Animation Loop  │◄──────┐
    │ - Update lights │       │
    │ - Rotate scene  │       │
    │ - Render frame  │───────┘
    └─────────────────┘
```

## Relationships

```
Scene
├── contains → PointCloud (Points)
│               └── uses → PointsNodeMaterial
│                           └── lightsNode → LightingModelContext
│                                             ├── lightingModel → CustomLightingModel
│                                             └── lights → [Light1, Light2, Light3]
├── contains → Light1 (PointLight)
│               └── child → Mesh (Sphere + NodeMaterial)
├── contains → Light2 (PointLight)
│               └── child → Mesh (Sphere + NodeMaterial)
└── contains → Light3 (PointLight)
                └── child → Mesh (Sphere + NodeMaterial)

Camera
└── controlled by → OrbitControls

Renderer (WebGPURenderer)
├── renders → Scene
└── uses → Camera
```
