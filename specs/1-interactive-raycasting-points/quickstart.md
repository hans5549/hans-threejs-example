# Quick Start: Three.js 互動式點雲 Raycasting 範例

**Feature**: `1-interactive-raycasting-points`  
**Date**: 2025-11-28

## 快速開始

### 1. 執行範例

```bash
# 進入專案目錄
cd examples/interactive-raycasting-points

# 使用任意 HTTP 伺服器開啟 (因為使用 ES Modules)
# 選項 A: Python
python3 -m http.server 8080

# 選項 B: Node.js (需安裝 serve)
npx serve .

# 選項 C: VS Code Live Server 擴充功能
# 右鍵點擊 index.html → Open with Live Server
```

然後在瀏覽器開啟: `http://localhost:8080`

### 2. 預期結果

開啟網頁後，你應該會看到：

1. **三組彩色點雲** - 紅色、綠色、青色，排列在畫面中央
2. **自動旋轉** - 場景會緩慢旋轉，展示 3D 效果
3. **FPS 面板** - 左上角顯示效能統計
4. **滑鼠互動** - 將滑鼠移到點雲上，會出現紅色球體標記

---

## 檔案結構

```
examples/interactive-raycasting-points/
└── index.html    # 單一檔案，包含所有 HTML/CSS/JS
```

---

## 關鍵程式碼導覽

### HTML 結構

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <title>Three.js - Interactive Raycasting Points</title>
  <style>
    /* 全螢幕畫布樣式 */
  </style>
</head>
<body>
  <div id="container"></div>
  <div id="info">three.js - interactive raycasting points</div>

  <!-- Import Map 配置 -->
  <script type="importmap">
    {
      "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
      }
    }
  </script>

  <!-- 主程式 -->
  <script type="module">
    // ... JavaScript 程式碼
  </script>
</body>
</html>
```

### 核心函數

| 函數 | 用途 |
|------|------|
| `init()` | 初始化場景、攝影機、渲染器、點雲 |
| `generatePointCloudGeometry()` | 產生點雲幾何資料 |
| `generatePointcloud()` | 建立標準點雲 |
| `generateIndexedPointcloud()` | 建立帶索引的點雲 |
| `generateIndexedWithOffsetPointcloud()` | 建立帶分組的點雲 |
| `onPointerMove()` | 處理滑鼠移動事件 |
| `onWindowResize()` | 處理視窗大小變更 |
| `animate()` | 動畫迴圈主函數 |
| `render()` | 渲染邏輯 (旋轉、raycasting、標記) |

---

## 自訂修改指南

### 修改點雲顏色

```javascript
// 在 init() 函數中找到這些行
const pcBuffer = generatePointcloud(new THREE.Color(1, 0, 0), width, length);  // 紅色
const pcIndexed = generateIndexedPointcloud(new THREE.Color(0, 1, 0), width, length);  // 綠色
const pcIndexedOffset = generateIndexedWithOffsetPointcloud(new THREE.Color(0, 1, 1), width, length);  // 青色

// 修改顏色參數 (R, G, B 範圍 0-1)
```

### 修改點雲密度

```javascript
// 在檔案頂部找到這些常數
const width = 80;   // 增加數值 = 更多點
const length = 160;
```

### 修改旋轉速度

```javascript
// 修改旋轉角度 (弧度)
const rotateY = new THREE.Matrix4().makeRotationY(0.005);  // 增加 = 更快
```

### 修改標記數量

```javascript
// 在 init() 函數中找到這行
for (let i = 0; i < 40; i++) {  // 修改 40 為其他數值
```

---

## 瀏覽器相容性

| 瀏覽器 | 最低版本 | 備註 |
|--------|----------|------|
| Chrome | 89+ | 完整支援 |
| Firefox | 89+ | 完整支援 |
| Safari | 15+ | 完整支援 |
| Edge | 89+ | 完整支援 |

**需求**: WebGL 2.0 支援

---

## 疑難排解

### 問題: 畫面空白

1. 確認使用 HTTP 伺服器開啟 (不要直接用 `file://`)
2. 檢查瀏覽器 Console 是否有錯誤訊息
3. 確認網路連線正常 (需要載入 CDN)

### 問題: FPS 很低

1. 確認沒有開啟其他耗資源的程式
2. 嘗試減少點雲密度 (降低 width/length)
3. 確認顯示卡驅動程式已更新

### 問題: 滑鼠互動沒反應

1. 確認滑鼠有移動到點雲區域
2. 嘗試增加 `threshold` 值 (預設 0.1)
3. 檢查 Console 是否有錯誤
