# Research: Unreal Bloom Post-processing Effect

**Feature**: 2-unreal-bloom  
**Date**: 2025-11-28  
**Status**: Complete

## Research Tasks

### 1. Three.js Post-processing Pipeline Architecture

**Task**: 研究 Three.js 後處理管線的最佳實踐

**Findings**:

Three.js 後處理使用 `EffectComposer` 作為核心，串聯多個 Pass 來處理渲染輸出：

```
Scene → RenderPass → UnrealBloomPass → OutputPass → Screen
```

**Decision**: 採用官方推薦的 Pass 組合
- `RenderPass`: 渲染基本場景
- `UnrealBloomPass`: 應用 Bloom 效果
- `OutputPass`: 最終輸出（處理色調映射和色彩空間轉換）

**Rationale**: 這是 Three.js 官方範例使用的標準組合，經過充分測試且效能最佳化。

**Alternatives Considered**:
- BloomPass（舊版）：效果較差，已被 UnrealBloomPass 取代
- 自訂 Shader：開發成本高，無必要

---

### 2. UnrealBloomPass Parameters

**Task**: 研究 UnrealBloomPass 的參數設定

**Findings**:

UnrealBloomPass 建構參數：
```javascript
new UnrealBloomPass(
  resolution,  // Vector2 - 渲染解析度
  strength,    // number - Bloom 強度 (預設 1.5)
  radius,      // number - Bloom 擴散半徑 (預設 0.4)
  threshold    // number - 亮度閾值 (預設 0.85)
)
```

可調整屬性：
- `threshold`: 決定多亮的像素會產生 Bloom
- `strength`: Bloom 效果的整體強度
- `radius`: Bloom 光暈的擴散範圍

**Decision**: 使用官方範例的參數配置
- resolution: `window.innerWidth, window.innerHeight`
- strength: 1.5 (初始化), 透過 GUI 可調 0-3
- radius: 0.4 (初始化), 透過 GUI 可調 0-1
- threshold: 0.85 (初始化), 透過 GUI 可調 0-1

**Rationale**: 官方預設值經過視覺效果調校，適合大多數場景。

---

### 3. GLTF Model Loading with Animation

**Task**: 研究 GLTF 模型載入和動畫播放

**Findings**:

GLTFLoader 載入流程：
```javascript
const loader = new GLTFLoader();
const gltf = await loader.loadAsync(url);
const model = gltf.scene;
const animations = gltf.animations;
```

AnimationMixer 使用：
```javascript
const mixer = new AnimationMixer(model);
const clip = animations[0];
mixer.clipAction(clip.optimize()).play();

// 在動畫迴圈中更新
mixer.update(deltaTime);
```

**Decision**: 
- 使用 `loadAsync` 進行非同步載入
- 使用 `clip.optimize()` 最佳化動畫效能
- 模型來源：`https://threejs.org/examples/models/gltf/PrimaryIonDrive.glb`

**Rationale**: 與官方範例一致，確保模型相容性。

---

### 4. OrbitControls Configuration

**Task**: 研究 OrbitControls 的配置選項

**Findings**:

關鍵限制參數：
```javascript
controls.maxPolarAngle = Math.PI * 0.5;  // 垂直旋轉上限（90度）
controls.minDistance = 3;                 // 最近縮放距離
controls.maxDistance = 8;                 // 最遠縮放距離
```

**Decision**: 採用規格定義的限制
- 垂直旋轉：0 - 90 度
- 縮放距離：3 - 8 單位

**Rationale**: 符合功能需求 FR-012、FR-013。

---

### 5. lil-gui Integration

**Task**: 研究 lil-gui 的使用模式

**Findings**:

基本使用：
```javascript
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';

const gui = new GUI();
const folder = gui.addFolder('bloom');
folder.add(params, 'threshold', 0.0, 1.0).onChange(callback);
```

**Decision**: 
- 建立 "bloom" 資料夾包含 threshold、strength、radius
- 建立 "tone mapping" 資料夾包含 exposure
- 預設展開所有資料夾

**Rationale**: 與官方範例結構一致，符合 FR-005 預設展開需求。

---

### 6. WebGL Detection and Fallback

**Task**: 研究 WebGL 偵測和降級策略

**Findings**:

WebGL 偵測方法：
```javascript
function isWebGLAvailable() {
  try {
    const canvas = document.createElement('canvas');
    return !!(window.WebGLRenderingContext && 
      (canvas.getContext('webgl') || canvas.getContext('experimental-webgl')));
  } catch (e) {
    return false;
  }
}
```

**Decision**: 
- 頁面載入時檢測 WebGL 支援
- 不支援時顯示 2D 降級版本（靜態說明頁面 + 截圖）

**Rationale**: 符合釐清問題的回答，提供基本的降級體驗。

---

### 7. Error Handling for Model Loading

**Task**: 研究模型載入錯誤處理

**Findings**:

錯誤處理模式：
```javascript
try {
  const gltf = await loader.loadAsync(url);
} catch (error) {
  showErrorUI(error);
}
```

**Decision**:
- 使用 try-catch 包裹載入邏輯
- 失敗時顯示錯誤訊息 + 重新載入按鈕
- 重試使用相同 URL，不實作備用來源

**Rationale**: 符合 Clarification 中的決策，簡單有效。

---

## Technology Stack Confirmation

| 元件 | 選擇 | 版本/來源 |
|------|------|----------|
| 核心引擎 | Three.js | CDN latest |
| 後處理 | EffectComposer + UnrealBloomPass | Three.js addons |
| 模型載入 | GLTFLoader | Three.js addons |
| 相機控制 | OrbitControls | Three.js addons |
| GUI | lil-gui | Three.js addons |
| 效能監控 | Stats | Three.js addons |
| 模組系統 | ES Modules + importmap | 瀏覽器原生 |

## Import Map Configuration

```json
{
  "imports": {
    "three": "https://unpkg.com/three@latest/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@latest/examples/jsm/"
  }
}
```

## Summary

所有技術問題已釐清，無需進一步研究。可直接進入 Phase 1 設計階段。
