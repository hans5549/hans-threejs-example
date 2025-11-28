# Research: WebGPU Volume Caustics

**Feature**: 4-webgpu-volume-caustics  
**Date**: 2025-11-28  
**Purpose**: 解決技術背景中的不確定項目並記錄技術決策

## Research Tasks

### 1. Three.js WebGPU 渲染器配置

**問題**: 如何正確初始化 WebGPURenderer 並處理不支援的情況？

**調查結果**:
- WebGPURenderer 是 Three.js r150+ 提供的 WebGPU 原生渲染器
- 需從 `three.webgpu.js` 匯入，非標準 `three.js`
- 初始化為非同步操作，需使用 `async/await`
- WebGPU 功能偵測：檢查 `navigator.gpu` 是否存在

**決策**: 使用 `three.webgpu.js` CDN 版本（r170+），初始化時先檢查 `navigator.gpu`

**備選方案**:
- 使用 npm 安裝 three 套件（較複雜，不適合單檔範例）
- 使用 WebGLRenderer 回退（放棄，不符合展示 WebGPU 的目的）

---

### 2. TSL (Three Shading Language) 焦散效果實作

**問題**: 如何使用 TSL 實作自訂焦散著色器？

**調查結果**:
- TSL 是基於節點的著色語言，取代傳統 GLSL
- 關鍵函式：`Fn()` 建立著色器函式、`uniform()` 建立 uniform 變數
- 焦散計算使用 `refract()` 計算折射向量
- 色散效果透過偏移 RGB 通道的 UV 座標實現（chromatic aberration）
- `castShadowNode` 用於自訂物體投射的陰影內容

**決策**: 使用官方範例的 TSL 焦散實作模式

**關鍵程式碼模式**:
```javascript
const causticEffect = Fn( () => {
    const refractionVector = refract( positionViewDirection.negate(), normalView, div( 1.0, ior ) ).normalize();
    const chromaticAberrationOffset = normalView.z.pow( - .9 ).mul( .004 );
    const causticProjection = vec3(
        texture( causticMap, textureUV.add( vec2( chromaticAberrationOffset.negate(), 0 ) ) ).r,
        texture( causticMap, textureUV.add( vec2( 0, chromaticAberrationOffset.negate() ) ) ).g,
        texture( causticMap, textureUV.add( vec2( chromaticAberrationOffset, chromaticAberrationOffset ) ) ).b
    );
    return causticProjection.mul( viewZ.mul( 60 ) ).add( viewZ ).mul( causticColor );
} )().toVar();
```

---

### 3. MeshPhysicalNodeMaterial 透明材質配置

**問題**: 如何配置玻璃透明材質以產生焦散效果？

**調查結果**:
- `MeshPhysicalNodeMaterial` 是支援 TSL 的物理材質
- 關鍵屬性：
  - `transmission: 1` - 完全透明
  - `thickness: 0.25` - 物體厚度（影響折射）
  - `ior: 1.5` - 折射率（玻璃約 1.5）
  - `roughness: 0.1` - 低粗糙度增強反射
  - `side: THREE.DoubleSide` - 雙面渲染
- `castShadowNode` 屬性用於自訂投射陰影（焦散效果）

**決策**: 使用 MeshPhysicalNodeMaterial 配合 castShadowNode 實現焦散

---

### 4. VolumeNodeMaterial 體積光實作

**問題**: 如何實作體積光（Volumetric Lighting）效果？

**調查結果**:
- `VolumeNodeMaterial` 用於體積渲染
- 需要 3D 噪聲紋理（Data3DTexture）模擬煙霧密度
- `scatteringNode` 定義散射函式，決定每個點的霧濃度
- 使用 `texture3D()` 採樣 3D 紋理
- 需要獨立的渲染 layer 和 pass 處理

**決策**: 
- 使用 ImprovedNoise 產生 3D 噪聲紋理
- 使用 layer 系統分離體積光渲染
- 體積光解析度降為 0.5 提升效能

**關鍵配置**:
```javascript
const LAYER_VOLUMETRIC_LIGHTING = 10;
volumetricMaterial.scatteringNode = Fn( ( { positionRay } ) => {
    const sampleGrain = ( scale, timeScale = 1 ) => 
        texture3D( noiseTexture3D, positionRay.add( timeScaled.mul( timeScale ) ).mul( scale ).mod( 1 ), 0 ).r.add( .5 );
    // ...
} );
```

---

### 5. 後處理 (Post Processing) 配置

**問題**: 如何配置後處理效果，包含 Bloom？

**調查結果**:
- Three.js WebGPU 使用 `PostProcessing` 類別
- `pass()` 建立場景渲染 pass
- `bloom()` 從 `three/addons/tsl/display/BloomNode.js` 匯入
- 多 pass 組合：scenePass + volumetricPass + bloomPass
- 體積光 pass 需設定 layer 過濾

**決策**: 使用三層 pass 架構：場景 → 體積光 → Bloom 合成

---

### 6. 聚光燈陰影配置

**問題**: 如何配置聚光燈以支援 HDR 焦散效果？

**調查結果**:
- SpotLight 需啟用 `castShadow: true`
- 關鍵配置：
  - `shadow.mapType: THREE.HalfFloatType` - HDR 陰影貼圖
  - `shadow.mapSize: 1024x1024` - 陰影解析度
  - `shadow.bias: -0.003` - 防止陰影瑕疵
  - `shadow.intensity: 0.95` - 陰影強度

**決策**: 使用 HalfFloatType 陰影貼圖支援 HDR 焦散

---

### 7. 資源載入策略

**問題**: 如何載入 GLTF 模型和紋理？

**調查結果**:
- GLTFLoader + DRACOLoader 載入壓縮 GLTF
- DRACO 解碼器路徑：`./jsm/libs/draco/gltf/`
- 模型：Three.js 官方 duck.glb
- 焦散紋理：opengameart/Caustic_Free.jpg
- 地板紋理：hardwood2_diffuse.jpg

**決策**: 使用 Three.js 官方資源路徑，CDN 載入

---

## Technology Best Practices

### WebGPU 錯誤處理
- 頁面載入時先檢查 `navigator.gpu`
- 提供清晰的瀏覽器不支援訊息
- 顯示靜態預覽圖片（根據 clarification）

### 效能優化
- 體積光 pass 解析度降為 0.5
- 使用 layer 系統分離渲染
- 避免每幀重新建立紋理

### 程式碼組織
- 遵循現有 examples/ 單檔結構
- 使用 ES Modules import map
- 清晰的初始化流程（async init）

## Resolved Clarifications

| 原不確定項目 | 解決方案 |
|-------------|---------|
| WebGPU 不支援處理 | 顯示靜態圖片 + 升級瀏覽器提示 |
| 低效能裝置處理 | 不特別處理，接受低幀率 |
| TSL 語法與用法 | 參考官方範例的 Fn/uniform/refract 模式 |
| 體積光實作方式 | VolumeNodeMaterial + scatteringNode + 3D noise |
| 資源載入路徑 | 使用 Three.js 官方 CDN 路徑 |
