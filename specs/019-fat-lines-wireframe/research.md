# Research: Fat Lines Wireframe

**Feature**: 019-fat-lines-wireframe  
**Date**: 2025-12-04  
**Purpose**: 解決技術上下文中的未知項目，並研究最佳實踐

## Research Tasks

### 1. Three.js Fat Lines 技術

**問題**: Fat Lines (LineMaterial, Wireframe, WireframeGeometry2) 的運作原理和使用方式

**發現**:
- Fat Lines 是 Three.js 的擴展功能，位於 `three/addons/lines/` 目錄
- 主要模組：
  - `LineMaterial`: 支援可變線寬的材質，使用 GPU 著色器實現
  - `Wireframe`: 用於渲染線框的特殊物件類別
  - `WireframeGeometry2`: 從任意幾何體生成線框幾何資料
- 與標準 `THREE.LineBasicMaterial` 的差異：
  - 標準線條受限於 WebGL 的 1 像素限制
  - Fat Lines 使用四邊形（quads）模擬粗線條，繞過 WebGL 限制
- `LineMaterial` 關鍵屬性：
  - `linewidth`: 像素單位的線寬（支援大於 1）
  - `dashed`: 啟用虛線效果
  - `dashScale`, `dashSize`, `gapSize`: 虛線參數
- 需要調用 `computeLineDistances()` 才能正確顯示虛線

**決策**: 使用官方 Three.js addons 中的 Fat Lines 模組

**理由**: 這是官方支援的解決方案，經過充分測試，且與 Three.js 核心版本同步更新

**考慮的替代方案**: 
- 自定義著色器實現 - 複雜度過高，維護困難
- 使用 `THREE.TubeGeometry` 建立管狀線條 - 效能較差，不適合線框渲染

---

### 2. 雙視口渲染技術

**問題**: 如何在單一畫布上渲染多個視口

**發現**:
- Three.js 支援 `renderer.setViewport(x, y, width, height)` 設定渲染區域
- 渲染流程：
  1. 設定主視口 `setViewport(0, 0, fullWidth, fullHeight)`
  2. 渲染主場景
  3. 設定插入視口 `setViewport(x, y, insetWidth, insetHeight)`
  4. 渲染相同場景（使用不同攝影機）
- 需要在每幀更新兩個攝影機的投影矩陣
- `setScissor` 可用於裁剪渲染區域，避免視口重疊問題

**決策**: 使用 setViewport 配合雙攝影機實現雙視口

**理由**: 這是 Three.js 標準做法，效能良好且易於實現

**考慮的替代方案**:
- 使用兩個獨立的 renderer - 資源浪費，同步困難
- 使用 CSS 分割畫面 - 無法共享 WebGL 上下文

---

### 3. GUI 控制面板最佳實踐

**問題**: lil-gui 的使用模式和參數綁定方式

**發現**:
- lil-gui 是 dat.GUI 的輕量級替代品
- 基本模式：
  ```javascript
  const gui = new GUI();
  gui.add(params, 'property', min, max).onChange(callback);
  gui.add(params, 'boolean').onChange(callback);
  gui.add(params, 'option', { 'Label': value }).onChange(callback);
  ```
- 支援的控制類型：
  - 數字滑桿（範圍）
  - 布林開關
  - 下拉選單
  - 顏色選擇器
- 回調函式在值變更時觸發

**決策**: 使用物件參數綁定配合 onChange 回調

**理由**: 提供即時反饋和清晰的程式碼結構

---

### 4. 虛線效果實現

**問題**: Fat Lines 的虛線效果需要特殊處理

**發現**:
- `LineMaterial.dashed` 是透過 shader defines 實現，不是 uniform
- 切換虛線需要：
  1. 設定 `matLine.dashed = true/false`
  2. 更新 defines：`matLine.defines.USE_DASH = ''` 或 `delete matLine.defines.USE_DASH`
  3. 設定 `matLine.needsUpdate = true` 觸發重新編譯
- `dashScale`, `dashSize`, `gapSize` 控制虛線外觀
- 需要先調用 `wireframe.computeLineDistances()` 計算線段距離

**決策**: 遵循官方範例的虛線切換模式

**理由**: 這是官方推薦的做法，確保相容性

---

### 5. 效能考量

**問題**: Fat Lines 的效能影響和最佳化策略

**發現**:
- Fat Lines 比標準線條消耗更多 GPU 資源（每條線段使用四邊形而非線段）
- Icosahedron (細分層級 1) 產生的線段數量適中，效能影響可接受
- 使用 `setAnimationLoop` 確保 60 fps 目標
- Stats.js 提供即時 FPS 監控

**決策**: 使用 Icosahedron (半徑 20, 細分 1) 作為展示幾何體

**理由**: 平衡視覺複雜度和效能，與官方範例一致

---

## 技術堆疊確認

| 項目 | 選擇 | 版本/來源 |
|------|------|----------|
| 3D 引擎 | Three.js | r174 (CDN) |
| Fat Lines | LineMaterial, Wireframe, WireframeGeometry2 | Three.js addons |
| GUI | lil-gui | Three.js addons |
| 效能監控 | Stats.js | Three.js addons |
| 軌道控制 | OrbitControls | Three.js addons |

## 風險與緩解

| 風險 | 可能性 | 影響 | 緩解策略 |
|------|--------|------|----------|
| 瀏覽器 WebGL 不支援 | 低 | 高 | 顯示友善錯誤訊息 |
| 虛線切換閃爍 | 中 | 低 | shader 重編譯是預期行為 |
| 視窗大小調整時比例失真 | 中 | 中 | 正確更新攝影機 aspect 和 renderer size |
