# Data Model: Unreal Bloom Post-processing Effect

**Feature**: 2-unreal-bloom  
**Date**: 2025-11-28  
**Status**: Complete

## Overview

本功能為純前端 3D 視覺化應用，無持久化資料需求。以下描述應用程式的執行時期資料結構和狀態管理。

## Application State

### Global State Object

```javascript
// 應用程式全域狀態
const appState = {
  // Three.js 核心物件
  scene: THREE.Scene,
  camera: THREE.PerspectiveCamera,
  renderer: THREE.WebGLRenderer,
  
  // 後處理
  composer: EffectComposer,
  bloomPass: UnrealBloomPass,
  
  // 動畫
  mixer: THREE.AnimationMixer,
  clock: THREE.Clock,
  
  // 控制
  controls: OrbitControls,
  
  // UI
  gui: GUI,
  stats: Stats
};
```

### Bloom Parameters

```javascript
// Bloom 效果參數物件（用於 GUI 綁定）
const params = {
  threshold: 0,      // 亮度閾值 [0.0 - 1.0]
  strength: 1,       // Bloom 強度 [0.0 - 3.0]
  radius: 0.5,       // 擴散半徑 [0.0 - 1.0]
  exposure: 1        // 曝光度 [0.1 - 2.0]
};
```

## Entity Definitions

### Scene（場景）

| 屬性 | 型別 | 說明 |
|------|------|------|
| children | Object3D[] | 場景中的所有 3D 物件 |
| background | Color/null | 背景顏色（本應用為 null/黑色） |

**包含內容**:
- 1x PerspectiveCamera（含 PointLight 子物件）
- 1x AmbientLight
- 1x 3D Model（Primary Ion Drive）

### Camera（相機）

| 屬性 | 型別 | 預設值 | 說明 |
|------|------|--------|------|
| fov | number | 40 | 視野角度 |
| aspect | number | window.innerWidth / window.innerHeight | 長寬比 |
| near | number | 1 | 近裁剪面 |
| far | number | 100 | 遠裁剪面 |
| position | Vector3 | (-5, 2.5, -3.5) | 初始位置 |

### Lights（燈光）

#### AmbientLight
| 屬性 | 型別 | 值 | 說明 |
|------|------|-----|------|
| color | Color | 0xcccccc | 環境光顏色 |

#### PointLight
| 屬性 | 型別 | 值 | 說明 |
|------|------|-----|------|
| color | Color | 0xffffff | 點光源顏色 |
| intensity | number | 100 | 光源強度 |
| parent | Object3D | camera | 跟隨相機移動 |

### 3D Model（模型）

| 屬性 | 型別 | 說明 |
|------|------|------|
| source | string | `models/gltf/PrimaryIonDrive.glb` |
| format | string | GLTF Binary (.glb) |
| scene | Group | 模型的 3D 場景圖 |
| animations | AnimationClip[] | 內建動畫列表 |

### Renderer（渲染器）

| 屬性 | 型別 | 預設值 | 說明 |
|------|------|--------|------|
| antialias | boolean | true | 抗鋸齒 |
| pixelRatio | number | window.devicePixelRatio | 像素密度 |
| toneMapping | ToneMapping | ReinhardToneMapping | 色調映射 |
| toneMappingExposure | number | Math.pow(exposure, 4.0) | 曝光度 |

### EffectComposer（效果合成器）

**Pass 順序**:
1. RenderPass - 渲染基礎場景
2. UnrealBloomPass - Bloom 後處理
3. OutputPass - 最終輸出

### UnrealBloomPass（Bloom 通道）

| 屬性 | 型別 | 預設值 | 範圍 | 說明 |
|------|------|--------|------|------|
| resolution | Vector2 | (width, height) | - | 渲染解析度 |
| strength | number | 1.5 | 0.0 - 3.0 | Bloom 強度 |
| radius | number | 0.4 | 0.0 - 1.0 | 擴散半徑 |
| threshold | number | 0.85 | 0.0 - 1.0 | 亮度閾值 |

### OrbitControls（軌道控制）

| 屬性 | 型別 | 值 | 說明 |
|------|------|-----|------|
| target | Vector3 | (0, 0, 0) | 旋轉中心點 |
| maxPolarAngle | number | Math.PI * 0.5 | 垂直旋轉上限 (90°) |
| minDistance | number | 3 | 最近縮放距離 |
| maxDistance | number | 8 | 最遠縮放距離 |

## Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                      User Interaction                        │
└─────────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
    ┌──────────┐        ┌──────────┐        ┌──────────┐
    │   GUI    │        │  Mouse   │        │  Window  │
    │  Change  │        │  Events  │        │  Resize  │
    └────┬─────┘        └────┬─────┘        └────┬─────┘
         │                   │                   │
         ▼                   ▼                   ▼
    ┌──────────┐        ┌──────────┐        ┌──────────┐
    │  params  │        │ controls │        │ renderer │
    │  update  │        │  update  │        │  resize  │
    └────┬─────┘        └────┬─────┘        └────┬─────┘
         │                   │                   │
         ▼                   ▼                   ▼
    ┌──────────┐        ┌──────────┐        ┌──────────┐
    │bloomPass │        │  camera  │        │ composer │
    │  update  │        │  update  │        │  resize  │
    └────┬─────┘        └────┬─────┘        └────┬─────┘
         │                   │                   │
         └───────────────────┼───────────────────┘
                             ▼
                    ┌─────────────────┐
                    │  Animation Loop │
                    │  (60 FPS max)   │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         ▼                   ▼                   ▼
    ┌──────────┐        ┌──────────┐        ┌──────────┐
    │  mixer   │        │  stats   │        │ composer │
    │  update  │        │  update  │        │  render  │
    └──────────┘        └──────────┘        └──────────┘
```

## State Transitions

### Application Lifecycle

```
[Initial] → [Loading] → [Ready] → [Running]
                ↓           ↓
            [Error]     [Resize]
                ↓           ↓
            [Retry] ←──────┘
```

### Loading States

| State | Description | UI Feedback |
|-------|-------------|-------------|
| Initial | 頁面載入中 | 無（空白頁面） |
| Loading | 模型載入中 | 可選：載入指示器 |
| Ready | 載入完成 | 顯示 3D 場景 |
| Error | 載入失敗 | 錯誤訊息 + 重試按鈕 |
| Running | 正常運行 | 動畫播放 + GUI 可用 |

## Validation Rules

### Parameter Constraints

| 參數 | 型別 | 最小值 | 最大值 | 步進 |
|------|------|--------|--------|------|
| threshold | float | 0.0 | 1.0 | any |
| strength | float | 0.0 | 3.0 | any |
| radius | float | 0.0 | 1.0 | 0.01 |
| exposure | float | 0.1 | 2.0 | any |

### Camera Constraints

| 約束 | 值 | 強制方式 |
|------|-----|----------|
| minDistance | 3 | OrbitControls.minDistance |
| maxDistance | 8 | OrbitControls.maxDistance |
| maxPolarAngle | π/2 | OrbitControls.maxPolarAngle |
