# Research: WebGL Geometry Terrain

**Feature**: 016-geometry-terrain
**Date**: 2025-12-04
**Status**: Complete

## Research Tasks

### 1. ImprovedNoise 演算法研究

**Decision**: 使用 Three.js 內建的 `ImprovedNoise` 類別

**Rationale**:
- Three.js 官方提供 `three/addons/math/ImprovedNoise.js`
- 基於 Ken Perlin 的改進噪聲演算法
- 產生連續、自然的高度變化
- 直接整合於 Three.js 生態系統，無需額外依賴

**Alternatives considered**:
- SimplexNoise: 效能略佳但視覺效果差異不大
- 自行實作 Perlin Noise: 增加複雜度，無明顯優勢
- 使用預生成高度圖圖片: 失去程序化生成的靈活性

### 2. 高度圖生成最佳實踐

**Decision**: 多層疊加 (Octave) 噪聲生成

**Rationale**:
- 使用多個頻率層疊加產生自然地形
- 低頻層產生大型山脈輪廓
- 高頻層增加細節和粗糙感
- 品質因子 (quality) 控制迭代次數

**Implementation Pattern**:
```javascript
// 偽程式碼
for (let j = 0; j < 4; j++) {
    for each point (x, z):
        height += perlin.noise(x/quality, z/quality, z) * quality
    quality *= 5
}
```

### 3. FirstPersonControls 使用模式

**Decision**: 使用 Three.js 官方 FirstPersonControls

**Rationale**:
- 官方維護，與 Three.js 版本同步
- 支援滑鼠視角控制和鍵盤移動
- 支援自訂移動速度和視角靈敏度
- 內建 `handleResize()` 方法處理視窗調整

**Configuration**:
- `movementSpeed`: 150 (每秒單位)
- `lookSpeed`: 0.1 (視角靈敏度)

### 4. Canvas 紋理生成技術

**Decision**: 使用 2D Canvas API 動態生成紋理

**Rationale**:
- 無需外部圖片資源
- 可根據高度資料即時計算明暗
- 支援添加噪點增加自然感
- 可升級後輸出為 CanvasTexture

**Implementation Pattern**:
1. 建立 Canvas 元素 (與高度圖同尺寸)
2. 計算每點的法向量
3. 根據光照方向計算明暗值
4. 添加隨機噪點
5. 升級至較大尺寸以提升紋理品質

### 5. WebGL 降級策略

**Decision**: 使用 WebGL 檢測 + 2D Canvas 備援

**Rationale**:
- 使用 `WebGLRenderer` 建構時的錯誤捕獲
- 降級時顯示靜態 2D 地形圖
- 提供使用者友善的提示訊息
- 保持基本視覺效果

**Detection Pattern**:
```javascript
try {
    renderer = new THREE.WebGLRenderer();
} catch (e) {
    showFallback();
}
```

### 6. 指數霧效果 (FogExp2) 參數

**Decision**: 使用 FogExp2 配合暖色調

**Rationale**:
- 指數霧比線性霧更自然
- 密度 0.0025 在 7500 單位場景中效果良好
- 霧色與背景色一致 (#efd1b5) 確保無縫融合

**Parameters**:
- Color: `0xefd1b5` (暖色調)
- Density: `0.0025`

### 7. 效能優化考量

**Decision**: 使用 Stats.js 監控 + setAnimationLoop

**Rationale**:
- `setAnimationLoop` 自動處理 requestAnimationFrame
- 背景標籤時自動暫停節省資源
- Stats.js 提供即時 FPS 監控
- Clock.getDelta() 確保時間差一致性

## Technology Stack Summary

| Component | Choice | Version/Source |
|-----------|--------|----------------|
| 3D Engine | Three.js | CDN (latest) |
| Noise Algorithm | ImprovedNoise | three/addons/math |
| Controls | FirstPersonControls | three/addons/controls |
| Performance Monitor | Stats.js | three/addons/libs |
| Texture Generation | Canvas 2D API | Browser native |

## Resolved Clarifications

All "NEEDS CLARIFICATION" items from Technical Context have been resolved:
- ✅ 演算法選擇: ImprovedNoise
- ✅ 控制器類型: FirstPersonControls  
- ✅ 紋理生成: Canvas 2D API
- ✅ 降級策略: 2D Canvas 備援
- ✅ 霧效果參數: FogExp2, density 0.0025
