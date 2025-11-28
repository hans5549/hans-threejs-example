# Quickstart: WebGPU Volume Caustics

**Feature**: 4-webgpu-volume-caustics  
**Date**: 2025-11-28

## Prerequisites

- 支援 WebGPU 的瀏覽器（Chrome 113+, Edge 113+）
- 本機 HTTP 伺服器（避免 CORS 問題）

## Quick Start

### 1. 檢查瀏覽器支援

開啟瀏覽器開發者工具 Console，輸入：
```javascript
navigator.gpu ? "WebGPU supported!" : "WebGPU NOT supported"
```

### 2. 啟動本機伺服器

在專案根目錄執行：

```bash
# 使用 Python 3
python -m http.server 8080

# 或使用 Node.js
npx serve .

# 或使用 VS Code Live Server 擴充套件
```

### 3. 開啟範例

瀏覽器開啟：
```
http://localhost:8080/examples/webgpu-volume-caustics/index.html
```

## 預期結果

✅ 看到一個透明的金色鴨子模型  
✅ 鴨子持續緩慢旋轉  
✅ 地面上顯示彩虹色焦散光斑  
✅ 可見霧氣中的體積光束  
✅ 可使用滑鼠旋轉和縮放視角  

## 互動操作

| 操作 | 效果 |
|------|------|
| 滑鼠左鍵拖曳 | 旋轉視角 |
| 滑鼠滾輪 | 縮放 |
| 觸控拖曳 | 旋轉視角 |
| 雙指縮放 | 縮放 |

## 故障排除

### WebGPU 不支援

**症狀**: 顯示靜態圖片和瀏覽器升級提示

**解決方案**:
1. 更新至 Chrome 113+ 或 Edge 113+
2. 確認 WebGPU 已啟用（chrome://flags → WebGPU）
3. 確認 GPU 驅動已更新

### 畫面黑屏

**可能原因**:
1. 模型載入失敗 - 檢查 Console 錯誤訊息
2. 著色器編譯錯誤 - 檢查 GPU 驅動版本

**解決方案**:
1. 開啟開發者工具 Network 頁籤，確認資源載入成功
2. 重新整理頁面

### 效能低落

**可能原因**: GPU 效能不足

**建議**:
1. 關閉其他 GPU 密集應用程式
2. 確認使用獨立顯卡（非內建顯示）
3. 本範例為展示用途，接受低幀率表現

## 檔案結構

```
examples/webgpu-volume-caustics/
└── index.html    # 完整的單頁應用
```

## 開發提示

### 修改焦散效果

焦散強度可在 `causticEffect` 函式中調整：
```javascript
// 調整焦散強度乘數（預設 60）
return causticProjection.mul( viewZ.mul( 60 ) )...
```

### 修改體積光濃度

透過 `smokeAmount` uniform 調整：
```javascript
const smokeAmount = uniform( 0.5 ); // 0-1 範圍
```

### 修改相機位置

```javascript
camera.position.set( x, y, z );
controls.target.set( 0, 0, 0 );
```

## 延伸閱讀

- [Three.js WebGPU 文件](https://threejs.org/docs/#manual/en/introduction/WebGPU)
- [TSL 著色語言 Wiki](https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language)
- [官方範例原始碼](https://github.com/mrdoob/three.js/blob/dev/examples/webgpu_volume_caustics.html)
