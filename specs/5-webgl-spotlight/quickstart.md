````markdown
# Quick Start: WebGL Spotlight 互動展示

**Feature**: `5-webgl-spotlight`  
**Date**: 2025-11-28

## 快速開始

### 1. 執行範例

```bash
# 進入專案目錄
cd examples/webgl-spotlight

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

1. **Lucy 雕像** - 場景中央的 3D 雕像模型
2. **動態光影** - 聚光燈自動環繞移動，產生動態陰影效果
3. **紋理投射** - 聚光燈投射出圖案紋理
4. **GUI 面板** - 右上角的控制面板
5. **軟陰影** - 地面上平滑的陰影效果

---

## 互動操作

### 攝影機控制

| 操作 | 效果 |
|------|------|
| 滑鼠左鍵拖曳 | 旋轉視角 |
| 滑鼠滾輪 | 縮放視角 (距離 2-10) |
| 滑鼠右鍵拖曳 | 平移視角 |

### GUI 控制面板

| 控制項 | 功能 | 範圍 |
|--------|------|------|
| map | 選擇紋理投射圖案 | none / disturb.jpg / colors.png / uv_grid_opengl.jpg |
| color | 調整光源顏色 | 顏色選擇器 |
| intensity | 調整光源強度 | 0 - 500 |
| distance | 調整光照距離限制 | 0 - 20 (0 = 無限) |
| angle | 調整聚光角度 | 0 - 60° |
| penumbra | 調整邊緣模糊度 | 0 - 1 |
| decay | 調整光照衰減率 | 1 - 2 |
| focus | 調整陰影聚焦 | 0 - 1 |
| shadowIntensity | 調整陰影強度 | 0 - 1 |
| helpers | 顯示/隱藏輔助線 | 開關 |

---

## 檔案結構

```
examples/webgl-spotlight/
├── index.html       # 主要 HTML 檔案，包含所有程式碼
└── fallback.png     # WebGL 不支援時的降級圖片
```

---

## 關鍵程式碼導覽

### HTML 結構

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <title>three.js webgl - lights - spotlight</title>
  <style>
    /* 全螢幕畫布樣式 */
  </style>
</head>
<body>
  <div id="info">three.js webgl - spotlight</div>

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
| `init()` | 初始化場景、攝影機、渲染器、光源、模型、GUI |
| `loadTextures()` | 載入紋理資源 |
| `createSpotLight()` | 建立並配置聚光燈 |
| `createGUI()` | 建立 GUI 控制面板 |
| `onWindowResize()` | 處理視窗大小變更 |
| `animate()` | 動畫迴圈主函數 |

---

## 自訂修改指南

### 修改聚光燈顏色

```javascript
// 在 init() 函數中找到 SpotLight 建立部分
spotLight = new THREE.SpotLight(0xff0000, 100);  // 紅色聚光燈
```

### 修改聚光燈移動速度

```javascript
// 在 animate() 函數中修改時間分母
const time = performance.now() / 1500;  // 加速 2 倍 (原本是 3000)
```

### 修改聚光燈移動半徑

```javascript
// 在 animate() 函數中修改半徑
spotLight.position.x = Math.cos(time) * 4;  // 半徑改為 4 (原本是 2.5)
spotLight.position.z = Math.sin(time) * 4;
```

### 修改攝影機距離限制

```javascript
// 在 controls 設定中修改
controls.minDistance = 1;   // 允許更近
controls.maxDistance = 20;  // 允許更遠
```

### 新增自訂紋理

```javascript
// 在 textures 物件中新增
const customTexture = loader.load('your-texture.jpg');
customTexture.minFilter = THREE.LinearFilter;
customTexture.magFilter = THREE.LinearFilter;
customTexture.generateMipmaps = false;
customTexture.colorSpace = THREE.SRGBColorSpace;
textures['your-texture.jpg'] = customTexture;
```

---

## 常見問題

### Q: 畫面全黑？

可能原因：
1. WebGL 不支援 - 檢查控制台是否有錯誤訊息
2. 紋理載入失敗 - 檢查網路連線
3. 模型載入失敗 - 應顯示後備球體

### Q: 模型不見了？

Lucy 模型從 Three.js CDN 載入，如果載入失敗會自動使用球體作為後備。檢查網路連線或 CDN 可用性。

### Q: GUI 面板太小？

可以透過 CSS 調整 lil-gui 的縮放：
```css
.lil-gui { transform: scale(1.2); transform-origin: top right; }
```

### Q: 陰影鋸齒嚴重？

調整陰影貼圖解析度：
```javascript
spotLight.shadow.mapSize.width = 2048;   // 原本是 1024
spotLight.shadow.mapSize.height = 2048;
```

---

## 效能優化建議

1. **降低陰影品質** - 如果效能不佳，可降低 `shadow.mapSize`
2. **減少模型面數** - 使用簡化的 Lucy 模型或後備球體
3. **關閉抗鋸齒** - 設定 `antialias: false`
4. **限制畫素比** - `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))`

---

## 外部資源

- [Three.js 官方文件 - SpotLight](https://threejs.org/docs/#api/en/lights/SpotLight)
- [Three.js 官方範例](https://threejs.org/examples/#webgl_lights_spotlight)
- [lil-gui 文件](https://lil-gui.georgealways.com/)

````
