````markdown
# Data Model: WebGL Spotlight 互動展示

**Feature**: `5-webgl-spotlight`  
**Date**: 2025-11-28  
**Status**: Complete

## 實體關係圖

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              Scene                                       │
│  (THREE.Scene - 包含所有 3D 物件的容器)                                  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
      ┌─────────────┬───────────────┼───────────────┬─────────────┐
      │             │               │               │             │
      ▼             ▼               ▼               ▼             ▼
┌───────────┐ ┌───────────┐ ┌─────────────┐ ┌───────────┐ ┌─────────────┐
│ SpotLight │ │ Hemisphere│ │   Ground    │ │ Lucy Model│ │   Helpers   │
│  (主光源)  │ │   Light   │ │   Plane     │ │ (3D 模型) │ │ (輔助視覺化)│
│           │ │  (環境光)  │ │  (地面)     │ │           │ │             │
└───────────┘ └───────────┘ └─────────────┘ └───────────┘ └─────────────┘
      │                                           │             │
      │                                           │             │
      ├─── map (紋理)                             │       ┌─────┴─────┐
      │                                           │       │           │
      ├─── shadow.camera ◄────────────────────────┤  SpotLight  CameraHelper
      │                                           │    Helper
      └─── lightHelper ◄──────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                            GUI Controls                                  │
│  (lil-gui - 使用者介面控制面板)                                          │
│                                                                          │
│  params: {                                                               │
│    map, color, intensity, distance, angle,                               │
│    penumbra, decay, focus, shadowIntensity, helpers                      │
│  }                                                                       │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                            即時更新 SpotLight 屬性
```

---

## 核心實體定義

### 1. Scene (場景)

| 屬性 | 類型 | 說明 |
|------|------|------|
| `children` | Object3D[] | 場景中的所有物件 |
| `background` | Color \| null | 背景顏色 (預設: null/黑色) |

### 2. Camera (透視攝影機)

| 屬性 | 類型 | 值 | 說明 |
|------|------|-----|------|
| `fov` | number | 40 | 視野角度 |
| `aspect` | number | window.innerWidth / window.innerHeight | 長寬比 |
| `near` | number | 0.1 | 近裁剪面 |
| `far` | number | 100 | 遠裁剪面 |
| `position` | Vector3 | (7, 4, 1) | 初始位置 |

### 3. SpotLight (聚光燈)

#### 基本屬性

| 屬性 | 類型 | 預設值 | 範圍 | 說明 |
|------|------|--------|------|------|
| `color` | Color | 0xffffff | - | 光源顏色 |
| `intensity` | number | 100 | 0-500 | 光源強度 (candela) |
| `distance` | number | 0 | 0-20 | 光照距離限制 (0=無限) |
| `angle` | number | π/6 | 0-π/3 | 聚光燈錐角 |
| `penumbra` | number | 1 | 0-1 | 邊緣模糊度 |
| `decay` | number | 2 | 1-2 | 光照衰減率 |
| `position` | Vector3 | (2.5, 5, 2.5) | - | 光源位置 (動態移動) |
| `map` | Texture | disturb.jpg | - | 紋理投射貼圖 |
| `name` | string | 'spotLight' | - | 物件名稱 |
| `castShadow` | boolean | true | - | 投射陰影 |

#### 陰影屬性 (shadow)

| 屬性 | 類型 | 值 | 說明 |
|------|------|-----|------|
| `mapSize.width` | number | 1024 | 陰影貼圖寬度 |
| `mapSize.height` | number | 1024 | 陰影貼圖高度 |
| `camera.near` | number | 2 | 陰影攝影機近平面 |
| `camera.far` | number | 10 | 陰影攝影機遠平面 |
| `focus` | number | 1 | 陰影聚焦 (0-1) |
| `bias` | number | -0.003 | 陰影偏移 |
| `intensity` | number | 1 | 陰影強度 (0-1) |

#### 附加輔助物件

| 屬性 | 類型 | 說明 |
|------|------|------|
| `lightHelper` | SpotLightHelper | 聚光燈輔助線 |
| `shadowCameraHelper` | CameraHelper | 陰影攝影機輔助線 |

### 4. HemisphereLight (環境光)

| 屬性 | 類型 | 值 | 說明 |
|------|------|-----|------|
| `skyColor` | Color | 0xffffff | 天空顏色 |
| `groundColor` | Color | 0x8d8d8d | 地面顏色 |
| `intensity` | number | 0.25 | 強度 |

### 5. Ground Plane (地面平面)

| 屬性 | 類型 | 值 | 說明 |
|------|------|-----|------|
| `geometry` | PlaneGeometry | (10, 10) | 平面幾何 |
| `material` | MeshLambertMaterial | { color: 0xbcbcbc } | 材質 |
| `position` | Vector3 | (0, -1, 0) | 位置 |
| `rotation.x` | number | -π/2 | 旋轉為水平 |
| `receiveShadow` | boolean | true | 接收陰影 |

### 6. Lucy Model (3D 模型)

| 屬性 | 類型 | 值 | 說明 |
|------|------|-----|------|
| `geometry` | BufferGeometry | PLY 載入 | Lucy 雕像幾何 |
| `material` | MeshLambertMaterial | {} | Lambert 材質 |
| `scale` | Vector3 | (0.0024, 0.0024, 0.0024) | 縮放比例 |
| `rotation.y` | number | -π/2 | Y 軸旋轉 |
| `position.y` | number | 0.8 | Y 軸位置 |
| `castShadow` | boolean | true | 投射陰影 |
| `receiveShadow` | boolean | true | 接收陰影 |

### 7. Fallback Model (後備幾何體)

> 當 Lucy 模型載入失敗時使用

| 屬性 | 類型 | 值 | 說明 |
|------|------|-----|------|
| `geometry` | SphereGeometry | (0.5, 32, 32) | 球體幾何 |
| `material` | MeshLambertMaterial | { color: 0xcccccc } | 材質 |
| `position.y` | number | 0.5 | Y 軸位置 |
| `castShadow` | boolean | true | 投射陰影 |
| `receiveShadow` | boolean | true | 接收陰影 |

### 8. Texture (紋理)

| 名稱 | 檔案 | 用途 |
|------|------|------|
| none | null | 無紋理 (純色光) |
| disturb.jpg | 擾動紋理 | 預設投射紋理 |
| colors.png | 彩色圖案 | 替代紋理選項 |
| uv_grid_opengl.jpg | UV 網格 | 替代紋理選項 |

#### 紋理屬性

| 屬性 | 值 | 說明 |
|------|-----|------|
| `minFilter` | LinearFilter | 縮小濾波 |
| `magFilter` | LinearFilter | 放大濾波 |
| `generateMipmaps` | false | 不產生 mipmap |
| `colorSpace` | SRGBColorSpace | sRGB 色彩空間 |

### 9. OrbitControls (軌道控制器)

| 屬性 | 類型 | 值 | 說明 |
|------|------|-----|------|
| `minDistance` | number | 2 | 最小縮放距離 |
| `maxDistance` | number | 10 | 最大縮放距離 |
| `maxPolarAngle` | number | π/2 | 最大極角 (不低於地平面) |
| `target` | Vector3 | (0, 1, 0) | 觀察目標點 |

### 10. Renderer (渲染器)

| 屬性 | 類型 | 值 | 說明 |
|------|------|-----|------|
| `antialias` | boolean | true | 抗鋸齒 |
| `toneMapping` | ToneMapping | NeutralToneMapping | 色調映射 |
| `toneMappingExposure` | number | 1 | 曝光度 |
| `shadowMap.enabled` | boolean | true | 啟用陰影 |
| `shadowMap.type` | ShadowMapType | PCFSoftShadowMap | 軟陰影 |

### 11. GUI Parameters (GUI 參數物件)

| 屬性 | 類型 | 預設值 | 控制項類型 | 說明 |
|------|------|--------|-----------|------|
| `map` | Texture | disturb.jpg | 下拉選單 | 紋理選擇 |
| `color` | number | 0xffffff | 顏色選擇器 | 光源顏色 |
| `intensity` | number | 100 | 滑桿 0-500 | 光源強度 |
| `distance` | number | 0 | 滑桿 0-20 | 光照距離 |
| `angle` | number | π/6 | 滑桿 0-π/3 | 聚光角度 |
| `penumbra` | number | 1 | 滑桿 0-1 | 邊緣模糊 |
| `decay` | number | 2 | 滑桿 1-2 | 衰減率 |
| `focus` | number | 1 | 滑桿 0-1 | 陰影聚焦 |
| `shadowIntensity` | number | 1 | 滑桿 0-1 | 陰影強度 |
| `helpers` | boolean | false | 開關 | 輔助線顯示 |

---

## 狀態轉換

### 動畫循環狀態

```
[初始化] → [載入資源] → [動畫循環]
                            │
                            ├── 更新聚光燈位置 (cos/sin)
                            ├── 更新輔助線
                            └── 渲染場景
```

### GUI 互動狀態

```
[GUI 變更] → [onChange 回調] → [更新 SpotLight 屬性] → [下一幀自動反映]
```

### 資源載入狀態

```
                    ┌─── [成功] → 建立 Lucy 網格
[載入 PLY 模型] ────┤
                    └─── [失敗] → 建立後備球體

                    ┌─── [成功] → 加入 textures 物件
[載入紋理] ─────────┤
                    └─── [失敗] → 使用 null (無紋理)
```

---

## 座標系統

### 世界座標

```
       Y (上)
       │
       │
       │
       └────── X (右)
      /
     /
    Z (前/攝影機方向)
```

### 關鍵位置

| 物件 | 位置 (x, y, z) | 說明 |
|------|----------------|------|
| 攝影機 | (7, 4, 1) | 初始觀察位置 |
| 聚光燈 | (2.5, 5, 2.5) → 動態 | 高度固定，XZ 平面圓形移動 |
| Lucy 模型 | (0, 0.8, 0) | 場景中心偏上 |
| 地面 | (0, -1, 0) | 場景底部 |
| 觀察目標 | (0, 1, 0) | 控制器觀察點 |

### 聚光燈移動軌跡

```
       Z
       │
   ────┼────
  /    │    \
 │  (0,5,0)  │  半徑: 2.5
  \    │    /   高度: 5 (固定)
   ────┼────    週期: ~18.85 秒 (2π × 3000ms)
       │
       X
```

````
