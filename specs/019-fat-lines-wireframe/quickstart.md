# Quickstart: Fat Lines Wireframe

**Feature**: 019-fat-lines-wireframe
**Date**: 2025-12-04
**Purpose**: 快速開始指南，說明如何執行和測試此功能

## 先決條件

- 現代瀏覽器（Chrome 90+, Firefox 88+, Edge 90+）
- WebGL 支援
- 本地 HTTP 伺服器（用於 ES modules）

## 快速啟動

### 方法 1: 使用 VS Code Live Server

1. 安裝 VS Code 擴充套件 "Live Server"
2. 右鍵點擊 `examples/webgl-lines-fat-wireframe/index.html`
3. 選擇 "Open with Live Server"
4. 瀏覽器將自動開啟範例頁面

### 方法 2: 使用 Python HTTP Server

```bash
# 在專案根目錄執行
cd examples/webgl-lines-fat-wireframe
python -m http.server 8000

# 開啟瀏覽器訪問
# http://localhost:8000
```

### 方法 3: 使用 Node.js http-server

```bash
# 安裝 http-server（如果尚未安裝）
npm install -g http-server

# 在專案根目錄執行
cd examples/webgl-lines-fat-wireframe
http-server -c-1

# 開啟瀏覽器訪問顯示的 URL
```

## 功能測試

### P1 功能測試（核心功能）

| 測試項目 | 步驟 | 預期結果 |
|----------|------|----------|
| 模型顯示 | 開啟頁面 | 藍色粗線條 Icosahedron 線框 |
| 軌道旋轉 | 滑鼠拖曳 | 模型視角旋轉 |
| 縮放控制 | 滾輪滾動 | 視圖放大/縮小 |

### P2 功能測試（互動控制）

| 測試項目 | 步驟 | 預期結果 |
|----------|------|----------|
| 線寬調整 | 拖曳 "width (px)" 滑桿 | 線條粗細即時變化 |
| 虛線切換 | 勾選 "dashed" | 線條變為虛線 |
| 虛線比例 | 調整 "dash / gap" | 虛線間隔變化 |
| 類型切換 | 選擇 "line type" | Fat Lines ↔ 標準線條 |

### P3 功能測試（進階功能）

| 測試項目 | 步驟 | 預期結果 |
|----------|------|----------|
| 插入視口 | 觀察右下角 | 顯示小型正方形視口 |
| 視窗調整 | 調整瀏覽器大小 | UI 元素正確重排 |

## GUI 控制說明

| 控制項 | 類型 | 範圍 | 說明 |
|--------|------|------|------|
| line type | 選擇 | 0, 1 | 0: Fat Lines, 1: gl.LINE |
| width (px) | 滑桿 | 1-10 | 線條寬度（像素） |
| dashed | 開關 | on/off | 虛線模式 |
| dash scale | 滑桿 | 0.5-1 | 虛線縮放 |
| dash / gap | 選擇 | 2:1, 1:1, 1:2 | 虛線與間隔比例 |

## 常見問題

### Q: 頁面顯示空白或錯誤

**A**: 檢查以下項目：

1. 確認使用 HTTP 伺服器（不能直接用 file:// 開啟）
2. 開啟瀏覽器開發者工具查看錯誤訊息
3. 確認瀏覽器支援 WebGL

### Q: 線條沒有顯示

**A**: 可能原因：

1. WebGL 未啟用 - 嘗試其他瀏覽器
2. GPU 驅動問題 - 更新顯示卡驅動

### Q: FPS 很低

**A**: Fat Lines 比標準線條消耗更多資源，嘗試：

1. 關閉其他瀏覽器分頁
2. 使用獨立顯示卡（如果有）
3. 降低瀏覽器視窗大小

## 檔案結構

```
examples/webgl-lines-fat-wireframe/
└── index.html    # 完整的範例程式碼（單一檔案）
```

## 依賴項

所有依賴項透過 CDN 載入，無需本地安裝：

- Three.js r174
- Three.js addons:
  - OrbitControls
  - LineMaterial
  - Wireframe
  - WireframeGeometry2
  - Stats
  - lil-gui
