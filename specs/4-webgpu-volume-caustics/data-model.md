# Data Model: WebGPU Volume Caustics

**Feature**: 4-webgpu-volume-caustics  
**Date**: 2025-11-28

## Overview

本功能為前端 3D 渲染應用，主要實體為 Three.js 場景物件。無後端資料庫，狀態完全在客戶端 JavaScript 中管理。

## Core Entities

### 1. Scene Configuration

場景的全域配置狀態。

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `LAYER_VOLUMETRIC_LIGHTING` | `number` | 體積光專用 layer ID | `10` |
| `camera.fov` | `number` | 相機視角（度） | `25` |
| `camera.near` | `number` | 近裁切面 | `0.025` |
| `camera.far` | `number` | 遠裁切面 | `5` |
| `camera.position` | `Vector3` | 相機初始位置 | `(-0.7, 0.2, 0.2)` |

### 2. SpotLight Configuration

聚光燈的配置，影響焦散效果品質。

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `position` | `Vector3` | 光源位置 | `(0.2, 0.3, 0.2)` |
| `intensity` | `number` | 光強度 | `1` |
| `angle` | `number` | 光錐角度 | `Math.PI / 6` |
| `penumbra` | `number` | 半影軟化 | `1` |
| `decay` | `number` | 衰減係數 | `2` |
| `castShadow` | `boolean` | 是否投射陰影 | `true` |
| `shadow.mapType` | `enum` | 陰影貼圖類型 | `THREE.HalfFloatType` |
| `shadow.mapSize` | `Vector2` | 陰影貼圖解析度 | `(1024, 1024)` |
| `shadow.bias` | `number` | 陰影偏移 | `-0.003` |
| `shadow.intensity` | `number` | 陰影強度 | `0.95` |

### 3. Duck Material (MeshPhysicalNodeMaterial)

透明玻璃鴨子的材質屬性。

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `color` | `Color` | 基底顏色 | `0xFFD700` (金色) |
| `transmission` | `number` | 透明度 (0-1) | `1` |
| `thickness` | `number` | 材質厚度 | `0.25` |
| `ior` | `number` | 折射率 | `1.5` |
| `metalness` | `number` | 金屬度 | `0` |
| `roughness` | `number` | 粗糙度 | `0.1` |
| `side` | `enum` | 渲染面 | `THREE.DoubleSide` |
| `transparent` | `boolean` | 啟用透明 | `true` |
| `castShadowNode` | `TSL Node` | 自訂焦散著色器 | (見下方) |
| `emissiveNode` | `TSL Node` | 自訂發光著色器 | (見下方) |

### 4. Caustic Effect Parameters

焦散效果的可調參數。

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `causticOcclusion` | `uniform(number)` | 遮蔽係數 | `1` |
| `chromaticAberrationOffset` | `computed` | 色散偏移量 | `normalView.z.pow(-0.9).mul(0.004)` |
| `causticIntensity` | `computed` | 焦散強度乘數 | `60` |

### 5. Volumetric Material (VolumeNodeMaterial)

體積光材質配置。

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `color` | `Color` | 體積光顏色 | (從光源繼承) |
| `scatteringNode` | `TSL Node` | 散射函式 | 3D noise based |
| `depthNode` | `TSL Node` | 深度遮蔽 | scene depth |

### 6. 3D Noise Texture Configuration

用於體積光的 3D 噪聲紋理。

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `size` | `number` | 紋理維度 (NxNxN) | `64` |
| `format` | `enum` | 紋理格式 | `THREE.RedFormat` |
| `type` | `enum` | 資料類型 | `THREE.UnsignedByteType` |
| `wrapS/T/R` | `enum` | 包裝模式 | `THREE.RepeatWrapping` |
| `minFilter/magFilter` | `enum` | 過濾模式 | `THREE.LinearFilter` |

### 7. Post Processing Configuration

後處理效果配置。

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `volumetricPass.resolutionScale` | `number` | 體積光解析度比例 | `0.5` |
| `bloomPass.strength` | `number` | Bloom 強度 | `1` |
| `bloomPass.radius` | `number` | Bloom 半徑 | `1` |
| `bloomPass.threshold` | `number` | Bloom 閾值 | `0` |
| `volumetricLightingIntensity` | `uniform(number)` | 體積光整體強度 | `1` |
| `smokeAmount` | `uniform(number)` | 煙霧濃度 | `0.5` |

### 8. Ground Plane

地面平面配置。

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `geometry` | `PlaneGeometry` | 幾何形狀 | `2 x 2` |
| `rotation.x` | `number` | X 軸旋轉 | `-Math.PI / 2` |
| `receiveShadow` | `boolean` | 接收陰影 | `true` |
| `material.color` | `Color` | 材質顏色 | `0x000000` |

## State Transitions

### Application Lifecycle

```
[Initial] → [Loading] → [Ready] → [Running]
                ↓           ↓
            [Error]    [Paused]
```

| State | Description | Triggers |
|-------|-------------|----------|
| Initial | 頁面載入前 | DOM ready |
| Loading | 載入資源中 | WebGPU check pass |
| Ready | 資源載入完成 | All assets loaded |
| Running | 動畫執行中 | renderer.setAnimationLoop |
| Paused | 暫停狀態 | Tab hidden (optional) |
| Error | 錯誤狀態 | WebGPU not supported / Load fail |

### WebGPU Support Check Flow

```
navigator.gpu exists?
    ├── Yes → Initialize WebGPURenderer
    └── No  → Show fallback image + message
```

## Validation Rules

### Material Constraints
- `transmission` must be in range [0, 1]
- `ior` must be positive (typically 1.0 - 2.5)
- `thickness` must be non-negative
- `roughness` must be in range [0, 1]

### Performance Constraints
- Shadow map size should be power of 2
- Noise texture size should be power of 2
- Volumetric resolution scale should be ≤ 1.0

### Camera Constraints
- `near` must be > 0 and < `far`
- `controls.maxDistance` limits zoom out
- Initial position should be within scene bounds

## External Resources

| Resource | Path | Description |
|----------|------|-------------|
| Duck Model | `models/gltf/duck.glb` | GLTF 鴨子模型 |
| Caustic Texture | `textures/opengameart/Caustic_Free.jpg` | 焦散投射紋理 |
| Floor Texture | `textures/hardwood2_diffuse.jpg` | 地板紋理 |
| DRACO Decoder | `jsm/libs/draco/gltf/` | GLTF 壓縮解碼器 |
