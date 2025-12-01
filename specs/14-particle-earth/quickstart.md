# Quickstart: Particle Earth Sphere

**Feature**: 14-particle-earth  
**Date**: 2025-12-01

## Prerequisites

- 現代瀏覽器（Chrome, Firefox, Safari, Edge）支援 WebGL
- 本地 HTTP 伺服器（用於開發，避免 CORS 問題）

## Quick Start

### 1. 啟動本地伺服器

```bash
# 進入專案目錄
cd hans-threejs-example

# 使用 Python 啟動簡易伺服器
python3 -m http.server 8080

# 或使用 Node.js
npx serve .
```

### 2. 開啟瀏覽器

```
http://localhost:8080/examples/particle-earth-sphere/
```

## File Structure

```
examples/particle-earth-sphere/
├── index.html           # 主程式（包含所有 JS 和 Shader）
└── assets/
    ├── earth-map.jpg    # 地球紋理（2048x1024）
    └── earth-fallback.jpg  # WebGL 降級圖片
```

## Controls

### 滑鼠互動

| 動作 | 效果 |
|------|------|
| 移動滑鼠 | 攝影機視角跟隨滑鼠平滑移動 |
| 滑鼠離開視窗 | 攝影機緩慢回到正面視角 |

### GUI 控制面板

| 控制項 | 說明 | 預設值 | 範圍 |
|--------|------|--------|------|
| Point Size | 粒子大小 | 2.0 | 1.0 - 10.0 |
| Rotation Speed | 自轉速度 | 0.001 | 0.0 - 0.01 |
| Opacity | 透明度 | 0.8 | 0.0 - 1.0 |

## Implementation Checklist

### Phase 1: 基礎架構
- [ ] 建立 HTML 結構
- [ ] 引入 Three.js 和 lil-gui
- [ ] 設定 Scene, Camera, Renderer

### Phase 2: 粒子系統
- [ ] 實作斐波那契球面分布演算法
- [ ] 建立 BufferGeometry
- [ ] 編寫 Vertex Shader
- [ ] 編寫 Fragment Shader
- [ ] 建立 ShaderMaterial

### Phase 3: 紋理和視覺
- [ ] 載入地球紋理
- [ ] 設定 UV 座標計算
- [ ] 套用加法混合模式

### Phase 4: 互動
- [ ] 實作滑鼠跟隨攝影機
- [ ] 實作滑鼠離開返回動畫
- [ ] 建立 GUI 控制面板

### Phase 5: 優化和降級
- [ ] 實作視窗 resize 處理
- [ ] 實作 WebGL 檢測和降級
- [ ] 效能測試和調優

## Troubleshooting

### 問題：畫面空白

1. 檢查瀏覽器 Console 是否有錯誤
2. 確認 WebGL 支援：`chrome://gpu/`
3. 確認紋理路徑正確

### 問題：效能低落

1. 減少粒子數量（修改 `numParticles`）
2. 降低 `pointSize`
3. 檢查 GPU 負載

### 問題：紋理不顯示

1. 確認使用 HTTP 伺服器（非 file:// 協議）
2. 檢查紋理檔案是否存在
3. 檢查 CORS 設定

## Development Tips

### 即時重載

使用 VS Code Live Server 或類似工具進行開發：

```bash
# 安裝 Live Server 擴充功能
# 右鍵 index.html > Open with Live Server
```

### Shader 偵錯

1. 使用 Spector.js 擴充功能檢視 WebGL 呼叫
2. 在 Shader 中輸出偵錯顏色：
   ```glsl
   gl_FragColor = vec4(vUv, 0.0, 1.0); // 顯示 UV 座標
   ```

### 效能監控

```javascript
import Stats from 'three/addons/libs/stats.module.js';
const stats = new Stats();
document.body.appendChild(stats.dom);
// 在 animate() 中呼叫 stats.update();
```
