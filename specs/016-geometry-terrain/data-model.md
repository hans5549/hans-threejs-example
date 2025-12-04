# Data Model: WebGL Geometry Terrain

**Feature**: 016-geometry-terrain
**Date**: 2025-12-04

## Entity Definitions

### 1. HeightMap

高度圖資料結構，儲存程序化生成的地形高度值。

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| data | Float32Array | 高度值陣列 | length = width × depth |
| width | number | 網格寬度 | 256 |
| depth | number | 網格深度 | 256 |

**Derivation**: 使用 ImprovedNoise 多層疊加生成，每個值範圍約 0-255。

### 2. TerrainGeometry

地形幾何資料，基於 PlaneGeometry 修改頂點高度。

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| geometry | THREE.PlaneGeometry | 平面幾何體 | segments = (width-1) × (depth-1) |
| physicalWidth | number | 物理寬度 | 7500 units |
| physicalDepth | number | 物理深度 | 7500 units |
| heightScale | number | 高度縮放因子 | 10 |

**Relationship**: 頂點 Y 座標 = HeightMap.data[i] × heightScale

### 3. TerrainTexture

地形紋理，動態生成基於高度和光照。

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| canvas | HTMLCanvasElement | 原始 Canvas | size = width × height |
| scaledCanvas | HTMLCanvasElement | 升級後 Canvas | size = width×2 × height×2 |
| texture | THREE.CanvasTexture | Three.js 紋理 | colorSpace = SRGBColorSpace |

**Generation Logic**:
1. 計算相鄰點高度差得出法向量
2. 與光源方向 (1,1,1) 點積計算明暗
3. 添加隨機噪點 (±5)
4. 升級至 2x 尺寸提升品質

### 4. SceneEnvironment

場景環境配置。

| Field | Type | Description | Value |
|-------|------|-------------|-------|
| backgroundColor | number | 背景色 | 0xefd1b5 |
| fogColor | number | 霧色 | 0xefd1b5 |
| fogDensity | number | 霧密度 | 0.0025 |
| fogType | string | 霧類型 | "FogExp2" |

### 5. CameraConfig

相機初始配置。

| Field | Type | Description | Value |
|-------|------|-------------|-------|
| fov | number | 視野角度 | 60 |
| near | number | 近裁切面 | 1 |
| far | number | 遠裁切面 | 10000 |
| initialPosition | Vector3 | 初始位置 | (100, 800, -800) |
| lookAt | Vector3 | 初始視向 | (-100, 810, -800) |

### 6. ControlsConfig

第一人稱控制器配置。

| Field | Type | Description | Value |
|-------|------|-------------|-------|
| movementSpeed | number | 移動速度 (units/sec) | 150 |
| lookSpeed | number | 視角靈敏度 | 0.1 |

## State Diagram

```
┌─────────────┐
│   初始化    │
└──────┬──────┘
       │
       ▼
┌─────────────┐     失敗      ┌─────────────┐
│ WebGL 檢測  │─────────────→│  降級顯示   │
└──────┬──────┘              └─────────────┘
       │ 成功
       ▼
┌─────────────┐
│ 生成高度圖  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 建立地形網格│
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 生成紋理    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 設定控制器  │
└──────┬──────┘
       │
       ▼
┌─────────────┐     resize    ┌─────────────┐
│ 動畫迴圈    │◄─────────────│ 視窗調整    │
└─────────────┘              └─────────────┘
```

## Data Flow

```
ImprovedNoise
     │
     ▼
┌─────────────┐
│  HeightMap  │
└──────┬──────┘
       │
       ├──────────────────┐
       ▼                  ▼
┌─────────────┐    ┌─────────────┐
│ TerrainMesh │    │TerrainTexture│
│ (vertices)  │    │ (colors)    │
└──────┬──────┘    └──────┬──────┘
       │                  │
       └────────┬─────────┘
                ▼
         ┌─────────────┐
         │    Mesh     │
         │ (geometry + │
         │  material)  │
         └──────┬──────┘
                │
                ▼
         ┌─────────────┐
         │    Scene    │
         └─────────────┘
```
