# API Contracts: WebGL Geometry Terrain

**Feature**: 016-geometry-terrain
**Date**: 2025-12-04

## Module Functions

此功能為純前端單一 HTML 範例，無 REST API。以下定義內部函式契約。

---

### generateHeight(width, height)

生成程序化高度圖資料。

**Parameters**:
| Name | Type | Description |
|------|------|-------------|
| width | number | 網格寬度 |
| height | number | 網格深度 |

**Returns**: `Uint8Array` - 高度值陣列 (length = width × height)

**Behavior**:
- 使用確定性種子確保可重現結果
- 多層噪聲疊加 (4 次迭代)
- 輸出值範圍: 0-255

---

### generateTexture(data, width, height)

根據高度資料生成地形紋理。

**Parameters**:
| Name | Type | Description |
|------|------|-------------|
| data | Uint8Array | 高度圖資料 |
| width | number | 紋理寬度 |
| height | number | 紋理高度 |

**Returns**: `HTMLCanvasElement` - 升級後的紋理 Canvas (2x 尺寸)

**Behavior**:
- 計算法向量基於相鄰高度差
- 光源方向: normalize(1, 1, 1)
- 添加隨機噪點 (±5)
- 輸出 Canvas 尺寸為輸入的 2 倍

---

### init()

初始化整個場景。

**Parameters**: 無

**Returns**: `void`

**Side Effects**:
- 建立 Scene, Camera, Renderer
- 生成高度圖和地形網格
- 設定 FirstPersonControls
- 啟動動畫迴圈
- 註冊視窗調整事件

**Error Handling**:
- WebGL 不支援時呼叫 showFallback()

---

### onWindowResize()

處理視窗尺寸變更。

**Parameters**: 無

**Returns**: `void`

**Behavior**:
- 檢查視窗尺寸是否低於最小閾值 (200x200)
- 更新相機 aspect ratio
- 更新渲染器尺寸
- 呼叫 controls.handleResize()

---

### animate()

動畫迴圈函式。

**Parameters**: 無

**Returns**: `void`

**Behavior**:
- 更新控制器 (基於 clock.getDelta())
- 渲染場景
- 更新 Stats

---

### showFallback()

WebGL 不支援時顯示降級內容。

**Parameters**: 無

**Returns**: `void`

**Behavior**:
- 隱藏 3D 容器
- 顯示 2D Canvas 靜態地形圖
- 顯示友善提示訊息
