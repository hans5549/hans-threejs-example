# Quickstart: COLLADA Kinematics 機器人手臂動畫

**Date**: 2025-11-28 | **Spec**: [spec.md](./spec.md)

## 快速開始

### 1. 專案結構

```
examples/collada-kinematics/
├── index.html           # 主程式（單一 HTML 檔案）
├── fallback.svg         # WebGL 降級圖片
└── models/              # 機器人模型目錄
    ├── abb_irb52_7_120.dae
    ├── kuka-kr5-r650.dae
    └── universalrobots-ur6-85-5-a.dae
```

### 2. 模型準備

#### ABB IRB 52（從 Three.js 官方取得）

```bash
# 下載 ABB 模型
curl -o models/abb_irb52_7_120.dae \
  "https://raw.githubusercontent.com/mrdoob/three.js/master/examples/models/collada/abb_irb52_7_120.dae"
```

#### KUKA KR5-R650（從 collada_robots 取得）

```bash
# 下載 .zae 檔案
curl -L -o kuka-kr5-r650.zae \
  "https://github.com/rdiankov/collada_robots/raw/master/kuka-kr5-r650.zae"

# 解壓縮（.zae 是 ZIP 格式）
unzip kuka-kr5-r650.zae -d models/

# 或在 macOS 上重命名後解壓
mv kuka-kr5-r650.zae kuka-kr5-r650.zip
unzip kuka-kr5-r650.zip -d models/
```

#### Universal Robots UR6（從 collada_robots 取得）

```bash
# 下載 .zae 檔案
curl -L -o universalrobots-ur6-85-5-a.zae \
  "https://github.com/rdiankov/collada_robots/raw/master/universalrobots-ur6-85-5-a.zae"

# 解壓縮
unzip universalrobots-ur6-85-5-a.zae -d models/
```

### 3. 本地開發伺服器

由於使用 ES Modules 和載入外部模型，需要 HTTP 伺服器：

```bash
# 方法 1: Python 3
cd examples/collada-kinematics
python3 -m http.server 8080

# 方法 2: Node.js (npx serve)
npx serve examples/collada-kinematics

# 方法 3: VS Code Live Server 擴充套件
# 右鍵 index.html → Open with Live Server
```

然後在瀏覽器開啟：`http://localhost:8080`

### 4. 技術相依

應用程式透過 CDN 載入以下套件（無需安裝）：

| 套件 | 版本 | 用途 |
|------|------|------|
| Three.js | r160 | 3D 渲染引擎 |
| ColladaLoader | (addon) | COLLADA 模型載入 |
| TWEEN.js | (addon) | 動畫補間引擎 |
| Stats.js | (addon) | FPS 監控面板 |

### 5. Import Map 設定

```html
<script type="importmap">
{
    "imports": {
        "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
    }
}
</script>
```

### 6. 使用說明

1. **查看機器人動畫**：頁面載入後自動播放 ABB IRB 52 機器人的關節動畫
2. **切換機器人**：使用右上角下拉選單選擇不同的機器人模型
3. **觀察 FPS**：左上角顯示即時 FPS 統計
4. **檢視機器人資訊**：左下角顯示當前機器人名稱和製造商

### 7. 瀏覽器相容性

| 瀏覽器 | 最低版本 | 備註 |
|--------|----------|------|
| Chrome | 89+ | 推薦 |
| Firefox | 78+ | |
| Safari | 15+ | |
| Edge | 89+ | |

**需求**: WebGL 1.0 支援（大多數現代瀏覽器皆支援）

### 8. 故障排除

#### 模型無法載入

- 確認 `models/` 目錄中有對應的 `.dae` 檔案
- 檢查瀏覽器控制台是否有 CORS 錯誤
- 確認使用 HTTP 伺服器而非直接開啟 HTML 檔案

#### 動畫不播放

- 確認模型包含運動學資訊
- 第三方模型可能缺少 kinematics 資料，此時會顯示靜態模型

#### 效能問題

- 確認不是在電池省電模式下執行
- 嘗試關閉其他消耗 GPU 的應用程式
- 使用 Chrome DevTools 的 Performance 面板分析

---

## 開發工作流程

1. **規格確認**: 閱讀 [spec.md](./spec.md) 了解完整需求
2. **資料模型**: 參考 [data-model.md](./data-model.md) 了解資料結構
3. **元件介面**: 參考 [contracts/component-interface.md](./contracts/component-interface.md) 了解 API
4. **任務分解**: 執行 `/speckit.tasks` 產生細部任務清單
5. **實作**: 依照任務清單逐步實作
6. **驗證**: 依照 spec.md 中的 Success Criteria 驗證功能
