# Quick Start Guide: WebGL Raycaster BVH

**Feature**: 8-raycaster-bvh  
**目標使用者**: 開發者、技術評估人員  
**預計閱讀時間**: 5 分鐘

---

## 什麼是 WebGL Raycaster BVH？

這是一個互動式 3D 展示應用程式，演示如何使用 **BVH (Bounding Volume Hierarchy)** 加速結構大幅提升光線追蹤性能。透過視覺化 100 條光線與 Stanford Bunny 模型的交點，使用者可以直接比較啟用/停用 BVH 時的性能差異。

### 核心功能
- ✅ FBX 3D 模型載入與渲染
- ✅ BVH 加速光線追蹤（60+ FPS vs 10-20 FPS）
- ✅ 即時性能監控與 FPS 警告
- ✅ 互動式 GUI 控制（光線數量、BVH 切換、視覺化）
- ✅ BVH 結構視覺化輔助器

---

## 快速開始

### 方式 1：直接開啟 HTML（推薦）

1. **下載檔案**
   ```bash
   # 複製檔案到你的專案
   cp examples/webgl-raycaster-bvh/index.html ./my-project/
   ```

2. **使用瀏覽器開啟**
   - 雙擊 `index.html`
   - 或使用本地伺服器（避免 CORS 問題）:
     ```bash
     # Python 3
     python -m http.server 8000
     
     # Node.js (http-server)
     npx http-server -p 8000
     ```

3. **訪問應用程式**
   - 開啟 `http://localhost:8000/index.html`
   - 等待模型載入（約 2-5 秒）

### 方式 2：整合到現有專案

#### 安裝相依套件

```bash
npm install three three-mesh-bvh
```

#### 基本程式碼範例

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { FBXLoader } from 'three/addons/loaders/FBXLoader.js';
import { computeBoundsTree, disposeBoundsTree, acceleratedRaycast } from 'three-mesh-bvh';

// 擴展 Three.js 原型
THREE.BufferGeometry.prototype.computeBoundsTree = computeBoundsTree;
THREE.BufferGeometry.prototype.disposeBoundsTree = disposeBoundsTree;
THREE.Mesh.prototype.raycast = acceleratedRaycast;

// 初始化場景
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 100);
camera.position.set(10, 10, 10);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 載入模型
const loader = new FBXLoader();
loader.load('path/to/model.fbx', (model) => {
    const mesh = model.children[0];
    
    // 建構 BVH
    mesh.geometry.computeBoundsTree();
    scene.add(mesh);
    
    // 執行光線追蹤
    const raycaster = new THREE.Raycaster();
    raycaster.set(new THREE.Vector3(5, 0, 0), new THREE.Vector3(-1, 0, 0));
    
    const intersects = raycaster.intersectObject(mesh);
    console.log('交點數量:', intersects.length);
});

// 渲染循環
function animate() {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
}
animate();
```

---

## 使用指南

### 初次載入

1. **觀察載入進度**
   - 畫面中央會顯示綠色進度條
   - 狀態訊息：「載入 FBX 模型...」→「建構 BVH...」→「準備完成」

2. **載入完成後**
   - 3D 場景顯示 Stanford Bunny 模型
   - 100 條白色光線從球面原點射向模型
   - 紅色球體標記交點位置
   - 左上角顯示 FPS 指標

### 互動控制

#### 滑鼠/觸控操作
- **旋轉視角**：左鍵拖曳
- **縮放**：滾輪或雙指縮放
- **平移**：右鍵拖曳（或 Ctrl + 左鍵）

#### GUI 控制面板（右上角）

| 控制項 | 功能 | 預設值 | 範圍 |
|--------|------|--------|------|
| **Ray Count** | 光線數量 | 100 | 1-200 |
| **Enable BVH** | 啟用 BVH 加速 | ✅ | - |
| **Show BVH Visualization** | 顯示 BVH 邊界框 | ❌ | - |
| **BVH Depth** | BVH 視覺化深度 | 10 | 1-20 |
| **Ray Opacity** | 光線透明度 | 0.5 | 0-1 |
| **Auto Rotate** | 自動旋轉模型 | ✅ | - |

---

## 性能測試指南

### 測試 1：BVH 加速效果驗證

**目標**：驗證 BVH 是否達成 70% 性能提升

**步驟**：
1. 開啟應用程式，等待載入完成
2. 觀察 FPS（左上角 Stats 指標）
3. 記錄「啟用 BVH」時的 FPS (預期: 60+)
4. **取消勾選「Enable BVH」**
5. 記錄「停用 BVH」時的 FPS (預期: 10-20)

**預期結果**：
```
啟用 BVH: 60 FPS
停用 BVH: 18 FPS
性能降級: 70% ✅
```

### 測試 2：光線數量性能曲線

**目標**：找出當前硬體的性能瓶頸

**步驟**：
1. 設定「Ray Count」為 10
2. 觀察 FPS，每次增加 10 條光線
3. 記錄 FPS < 60 時的光線數量

**範例結果**：
| 光線數量 | FPS (BVH ON) | FPS (BVH OFF) |
|---------|-------------|---------------|
| 10 | 60 | 45 |
| 50 | 60 | 22 |
| 100 | 60 | 12 |
| 150 | 58 | 8 |
| 200 | 52 | 5 |

### 測試 3：BVH 視覺化探索

**目標**：理解 BVH 樹狀結構

**步驟**：
1. 勾選「Show BVH Visualization」
2. 調整「BVH Depth」滑桿 (1-20)
3. 觀察邊界框的層級結構
   - 深度 1: 最頂層大框
   - 深度 10: 中等細節
   - 深度 20: 最細緻的葉節點

**觀察重點**：
- 淺層邊界框覆蓋大範圍
- 深層邊界框緊密包覆模型細節
- 光線首先與大框測試（快速剔除）

---

## 常見問題排解

### Q1: 模型載入超時（超過 60 秒）

**症狀**：
- 進度條停在某個百分比
- 顯示「模型載入超時」錯誤訊息

**解決方案**：
1. **檢查網路連線**
   - 模型從 Three.js CDN 載入（約 2-3 MB）
   - 測試網址：https://threejs.org/examples/models/fbx/stanford-bunny.fbx

2. **使用本地伺服器**
   ```bash
   python -m http.server 8000
   ```

3. **檢查瀏覽器控制台**
   - 開啟開發者工具（F12）
   - 查看 Console 頁籤的錯誤訊息

### Q2: FPS 低於 30（出現黃色警告）

**症狀**：
- 畫面上方顯示「⚠️ FPS < 30 - 考慮減少光線數量」

**解決方案**：
1. **降低光線數量**
   - 將「Ray Count」調整為 50 或更低
   
2. **確認 BVH 已啟用**
   - 檢查「Enable BVH」勾選狀態
   
3. **關閉 BVH 視覺化**
   - 取消勾選「Show BVH Visualization」（額外渲染負擔）

4. **檢查硬體加速**
   - Chrome: `chrome://gpu/`
   - 確認 WebGL 正常啟用

### Q3: 光線沒有顯示

**可能原因**：
- 光線透明度設定為 0
- 模型尺寸問題導致光線在視野外

**解決方案**：
1. **調整光線透明度**
   - 將「Ray Opacity」設定為 0.5-1.0

2. **重設攝影機視角**
   - 重新整理頁面
   - 使用滑鼠縮放到合適距離

### Q4: BVH 視覺化看起來很亂

**這是正常現象！**
- BVH 深度越高，邊界框數量越多
- 建議從深度 1 開始逐步增加

**最佳觀察深度**：
- 深度 1-3: 觀察整體結構
- 深度 5-10: 平衡細節與清晰度
- 深度 15+: 研究葉節點細節

### Q5: 在行動裝置上性能很差

**預期行為**：
- 本應用程式針對桌面瀏覽器最佳化
- 行動裝置 GPU 性能有限

**優化建議**：
1. 降低光線數量至 30-50
2. 關閉 BVH 視覺化
3. 停用「Auto Rotate」
4. 使用較新的旗艦級行動裝置

---

## 技術細節

### 系統需求

| 項目 | 最低需求 | 推薦配置 |
|------|---------|---------|
| **瀏覽器** | Chrome 90+ / Firefox 88+ / Safari 14+ | Chrome 最新版 |
| **GPU** | 支援 WebGL 2.0 | 獨立顯卡 |
| **RAM** | 4 GB | 8 GB+ |
| **網路** | 1 Mbps | 10 Mbps+ |

### 相依套件版本

```json
{
  "three": "^0.160.0",
  "three-mesh-bvh": "^0.7.3"
}
```

### 關鍵性能指標

| 指標 | 目標值 | 實際值（範例）|
|-----|--------|------------|
| 模型載入時間 | <5s | 2.3s |
| BVH 建構時間 | <500ms | 312ms |
| FPS（100 rays, BVH ON） | ≥60 | 62 |
| FPS（100 rays, BVH OFF） | 10-20 | 18 |
| 記憶體使用 | <50 MB | 26 MB |

---

## 進階使用

### 自訂模型

替換為你自己的 FBX 模型：

```javascript
// 修改 loadModel 函式呼叫
loadModel('path/to/your-model.fbx')
    .then(model => {
        // 模型載入完成
    });
```

**注意事項**：
- 模型三角形數量建議 10k-100k（性能平衡）
- 過於複雜的模型會增加 BVH 建構時間
- 確保模型包含有效的幾何資料

### 調整 BVH 參數

```javascript
buildBVH(mesh, {
    strategy: 'SAH',     // 改用 SAH 策略（更優但更慢）
    maxDepth: 30,        // 降低深度（減少記憶體）
    maxLeafTris: 5       // 降低葉節點三角形數（更細緻）
});
```

**策略比較**：
- **CENTER**：最快建構，良好性能
- **AVERAGE**：平衡建構速度與查詢性能
- **SAH**：最佳查詢性能，但建構慢

### 修改光線分布

```javascript
// 改用網格分布而非球面分布
function generateGridOrigins(gridSize, spacing, height) {
    const origins = [];
    const offset = (gridSize - 1) * spacing / 2;
    
    for (let i = 0; i < gridSize; i++) {
        for (let j = 0; j < gridSize; j++) {
            const x = i * spacing - offset;
            const z = j * spacing - offset;
            origins.push(new THREE.Vector3(x, height, z));
        }
    }
    
    return origins;
}

// 使用：產生 10x10 網格，間距 1，高度 5
const origins = generateGridOrigins(10, 1, 5);
```

---

## 學習資源

### 相關範例
- [Three.js WebGL Raycaster BVH 官方範例](https://threejs.org/examples/#webgl_raycaster_bvh)
- [three-mesh-bvh 光線追蹤範例](https://gkjohnson.github.io/three-mesh-bvh/example/bundle/raycast.html)

### 文件連結
- [Three.js 文件](https://threejs.org/docs/)
- [three-mesh-bvh GitHub](https://github.com/gkjohnson/three-mesh-bvh)
- [BVH 演算法原理（Wikipedia）](https://en.wikipedia.org/wiki/Bounding_volume_hierarchy)

### 影片教學
- [Three.js 光線追蹤基礎](https://www.youtube.com/results?search_query=threejs+raycasting)
- [BVH 加速結構講解](https://www.youtube.com/results?search_query=bvh+raytracing)

---

## 下一步

現在你已經了解基本使用方式，可以：

1. **實驗不同參數**
   - 嘗試不同的光線數量
   - 觀察 BVH 視覺化不同深度
   - 測試性能極限

2. **整合到專案**
   - 將 BVH 技術應用到你的 3D 應用
   - 優化大型場景的光線追蹤性能

3. **深入學習**
   - 研究 BVH 演算法原理
   - 探索 three-mesh-bvh 進階功能（距離查詢、形狀投射等）

4. **貢獻改進**
   - 回報問題或建議
   - 提交改進 Pull Request

---

## 需要協助？

- **檢視完整規格文件**: `specs/8-raycaster-bvh/spec.md`
- **查看資料模型**: `specs/8-raycaster-bvh/data-model.md`
- **研究技術細節**: `specs/8-raycaster-bvh/research.md`
- **API 介面說明**: `specs/8-raycaster-bvh/contracts/component-interface.md`

---

**祝你探索愉快！** 🚀
