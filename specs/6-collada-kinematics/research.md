# Research: COLLADA Kinematics 機器人手臂動畫

**Date**: 2025-11-28 | **Spec**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md)

## 研究摘要

本文件記錄 Phase 0 研究階段的所有發現，解決 Technical Context 中標記為 "NEEDS CLARIFICATION" 的項目。

---

## 1. 機器人模型來源與格式

### 1.1 ABB IRB 52 模型

**Decision**: 使用 Three.js 官方範例提供的模型

**Rationale**: 
- 模型位於 `three.js/examples/models/collada/abb_irb52_7_120.dae`
- 已經過 Three.js ColladaLoader 驗證
- 包含完整的運動學資訊（joints, kinematics）
- 無需額外處理即可使用

**Source URL**: 
```
https://raw.githubusercontent.com/mrdoob/three.js/refs/heads/master/examples/models/collada/abb_irb52_7_120.dae
```

### 1.2 KUKA KR5-R650 模型

**Decision**: 從 collada_robots 倉庫下載並解壓縮 .zae 檔案

**Rationale**:
- 模型存在於 `rdiankov/collada_robots` 倉庫
- 檔案格式為 .zae（壓縮的 COLLADA 檔案）
- 需要解壓縮為 .dae 格式才能使用

**Source URL**:
```
https://github.com/rdiankov/collada_robots/raw/master/kuka-kr5-r650.zae
```

**Alternatives Considered**:
- ❌ 直接載入 .zae：ColladaLoader 不支援
- ❌ 使用其他 KUKA 模型（kr150, kr360）：KR5-R650 體積適中，適合展示

### 1.3 Universal Robots UR6 模型

**Decision**: 從 collada_robots 倉庫下載並解壓縮 .zae 檔案

**Rationale**:
- 模型存在於 `rdiankov/collada_robots` 倉庫
- 檔案名稱：`universalrobots-ur6-85-5-a.zae`
- 需要解壓縮為 .dae 格式

**Source URL**:
```
https://github.com/rdiankov/collada_robots/raw/master/universalrobots-ur6-85-5-a.zae
```

---

## 2. .zae 檔案解壓縮流程

### 2.1 什麼是 .zae 格式

**Finding**: .zae 是 COLLADA 的壓縮封存格式，本質上是一個 ZIP 檔案，包含：
- `.dae` 主要模型檔案
- 相關的紋理和材質檔案
- 參考資源

### 2.2 解壓縮步驟

**Decision**: 使用標準解壓縮工具處理

**Steps**:
```bash
# 方法 1: 使用 unzip
unzip kuka-kr5-r650.zae -d kuka-kr5-r650/

# 方法 2: 重命名後解壓
mv kuka-kr5-r650.zae kuka-kr5-r650.zip
unzip kuka-kr5-r650.zip

# 方法 3: macOS 可直接解壓
# 雙擊 .zae 檔案（可能需要先重命名為 .zip）
```

**Expected Output**:
```
kuka-kr5-r650/
├── kuka-kr5-r650.dae    # 主要模型檔案
├── textures/            # 紋理資料夾（如果有）
└── ...
```

### 2.3 注意事項

- 解壓後需要檢查 .dae 檔案內部的相對路徑參照
- 如果有紋理檔案，需要確保路徑正確
- 部分模型可能只包含幾何資料，無紋理

---

## 3. ColladaLoader 相容性

### 3.1 Three.js ColladaLoader 功能

**Finding**: ColladaLoader 支援以下 COLLADA 特性：
- ✅ 幾何資料（geometry）
- ✅ 材質（materials）
- ✅ 骨架和關節（joints, kinematics）
- ✅ 動畫（animations）
- ✅ 場景層級（scene hierarchy）
- ⚠️ 部分 COLLADA 1.5 擴展功能

### 3.2 運動學資料結構

**Finding**: ColladaLoader 解析後的運動學資料結構：

```javascript
collada.kinematics.joints // 關節陣列
// 每個關節包含：
{
  static: boolean,           // 是否為靜態關節
  limits: { min, max },      // 運動範圍限制
  // ...其他運動學參數
}
```

### 3.3 相容性風險

**Risk Assessment**:
| 模型 | 風險等級 | 說明 |
|------|----------|------|
| ABB IRB 52 | 低 | Three.js 官方範例使用，已驗證 |
| KUKA KR5-R650 | 中 | 第三方模型，需測試運動學資料 |
| UR6 | 中 | 第三方模型，需測試運動學資料 |

**Mitigation**:
- 載入時檢查 `kinematics.joints` 是否存在
- 若缺少運動學資訊，顯示靜態模型並通知使用者

---

## 4. 參考實作分析

### 4.1 Three.js 官方範例結構

**Source**: `examples/webgl_loader_collada_kinematics.html`

**Key Implementation Patterns**:

```javascript
// 1. 載入模型
const loader = new ColladaLoader();
loader.load(modelUrl, function(collada) {
    dae = collada.scene;
    kinematics = collada.kinematics;
    scene.add(dae);
    
    // 2. 初始化關節動畫
    setupTween();
});

// 3. TWEEN 動畫設定
function setupTween() {
    const duration = THREE.MathUtils.randInt(1000, 5000);
    const target = {};
    
    for (const prop in kinematics.joints) {
        if (kinematics.joints[prop].static !== true) {
            const joint = kinematics.joints[prop];
            target[prop] = THREE.MathUtils.randFloat(joint.limits.min, joint.limits.max);
        }
    }
    
    tweenParameters = kinematicsModel.setJointValue;
    
    new TWEEN.Tween(tweenParameters)
        .to(target, duration)
        .easing(TWEEN.Easing.Quadratic.Out)
        .start();
}
```

### 4.2 關鍵技術要點

1. **運動學設定**: `kinematics.setJointValue(jointName, value)` 設定關節角度
2. **動畫循環**: TWEEN 動畫完成後自動重新觸發 `setupTween()`
3. **限制範圍**: 使用 `joint.limits.min/max` 確保關節在有效範圍內

---

## 5. 專案架構決策

### 5.1 檔案結構

**Decision**: 採用單一 HTML 檔案模式

**Rationale**:
- 與現有專案範例（webgl-spotlight, css3d-periodic-table）一致
- 便於部署和分享
- 減少複雜度

**Structure**:
```
examples/collada-kinematics/
├── index.html           # 主程式（HTML + CSS + JS 合一）
├── fallback.svg         # WebGL 降級圖片
└── models/              # 模型檔案目錄
    ├── abb_irb52_7_120.dae
    ├── kuka-kr5-r650.dae
    └── universalrobots-ur6-85-5-a.dae
```

### 5.2 模型檔案託管

**Decision**: 將模型檔案放在專案本地

**Rationale**:
- 避免 CORS 問題
- 確保模型可用性
- 便於版本控制

**Alternatives Considered**:
- ❌ 使用 CDN：CORS 限制、可用性不可控
- ❌ 使用 Three.js 官方 CDN：僅有 ABB 模型

### 5.3 CDN 依賴

**Decision**: 使用 unpkg.com 載入 Three.js 及其 addons

**Import Map**:
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

---

## 6. UI 設計決策

### 6.1 機器人選擇器

**Decision**: 右上角下拉選單

**Rationale**:
- 符合使用者慣例（設定選項通常在右上角）
- 不遮擋主要 3D 內容
- 與 FPS 監控面板（左上角）對稱

**Implementation**:
```html
<select id="robotSelector" style="position: absolute; top: 10px; right: 10px;">
    <option value="abb">ABB IRB 52</option>
    <option value="kuka">KUKA KR5-R650</option>
    <option value="ur6">Universal Robots UR6</option>
</select>
```

### 6.2 載入進度指示器

**Decision**: 螢幕中央顯示載入百分比

**Rationale**:
- 明確的視覺回饋
- 不依賴額外 UI 庫
- 載入完成後自動隱藏

**Implementation**:
```html
<div id="loadingOverlay" style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);">
    <div id="loadingProgress">Loading... 0%</div>
</div>
```

---

## 7. 風險緩解策略

### 7.1 模型載入失敗

**Strategy**: 提供降級處理
```javascript
loader.load(modelUrl, 
    onLoad,
    onProgress,
    function(error) {
        console.error('Model loading failed:', error);
        showFallbackMessage('無法載入模型，請檢查網路連線');
    }
);
```

### 7.2 缺少運動學資訊

**Strategy**: 顯示靜態模型
```javascript
if (!collada.kinematics || Object.keys(collada.kinematics.joints).length === 0) {
    console.warn('No kinematics data found');
    showStaticModel(collada.scene);
}
```

### 7.3 WebGL 不支援

**Strategy**: 顯示 fallback.svg
```javascript
if (!WEBGL.isWebGLAvailable()) {
    document.body.appendChild(WEBGL.getWebGLErrorMessage());
    return;
}
```

---

## 8. 結論

所有 "NEEDS CLARIFICATION" 項目已解決：

| 項目 | 狀態 | 決策 |
|------|------|------|
| 模型檔案可取得 | ✅ 已解決 | ABB 用 Three.js 官方；KUKA/UR 從 collada_robots 解壓 |
| .zae 解壓流程 | ✅ 已解決 | 使用 unzip 或重命名為 .zip 後解壓 |
| ColladaLoader 相容性 | ✅ 已解決 | 支援主要功能，需測試第三方模型 |
| 專案架構 | ✅ 已解決 | 單一 HTML + 本地模型檔案 |

**Next Step**: 進入 Phase 1，建立 data-model.md 和 contracts/
