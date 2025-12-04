# Research: WebGL Effects ASCII

**Feature**: 017-effects-ascii  
**Date**: 2025-01-27  
**Status**: Complete

## Overview

本文件記錄 AsciiEffect 和相關 Three.js API 的研究結果，解決 Technical Context 中的所有技術問題。

---

## 1. AsciiEffect API 研究

### Decision
使用 Three.js 內建的 `AsciiEffect` 類別，透過 CDN 從 `three/addons/effects/AsciiEffect.js` 載入。

### Rationale
- AsciiEffect 是 Three.js 官方維護的 addon，穩定且與 Three.js 版本同步
- 無需額外安裝，透過 importmap 直接使用
- 支援豐富的自訂選項（字元集、反轉、顏色等）

### API 詳細說明

#### Constructor
```javascript
new AsciiEffect(renderer, charSet, options)
```

**Parameters**:
| 參數 | 類型 | 說明 |
|------|------|------|
| `renderer` | WebGLRenderer | Three.js WebGL 渲染器實例 |
| `charSet` | string | ASCII 字元集，從暗到亮排列 |
| `options` | object | 可選配置選項 |

#### Options 物件
| 選項 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| `resolution` | number | 0.15 | 字元解析度（較小值 = 更多字元） |
| `scale` | number | 1 | 字元縮放比例 |
| `color` | boolean | false | 是否保留原始顏色 |
| `alpha` | boolean | false | 是否保留透明度 |
| `block` | boolean | false | 是否使用區塊字元模式 |
| `invert` | boolean | false | 反轉字元順序（白字黑底） |
| `strResolution` | string | 'low' | 字元解析度預設值 ('low', 'medium', 'high') |

#### Properties
| 屬性 | 類型 | 說明 |
|------|------|------|
| `domElement` | HTMLDivElement | 包含 ASCII 渲染結果的 DOM 元素 |

#### Methods
| 方法 | 參數 | 說明 |
|------|------|------|
| `setSize(width, height)` | number, number | 設定渲染尺寸 |
| `render(scene, camera)` | Scene, Camera | 渲染場景到 ASCII |

### 實作範例
```javascript
import { AsciiEffect } from 'three/addons/effects/AsciiEffect.js';

// 建立效果器
const effect = new AsciiEffect(renderer, ' .:-+*=%@#', { invert: true });
effect.setSize(window.innerWidth, window.innerHeight);

// 設定樣式
effect.domElement.style.color = 'white';
effect.domElement.style.backgroundColor = 'black';

// 添加到 DOM
document.body.appendChild(effect.domElement);

// 渲染
effect.render(scene, camera);
```

### Alternatives Considered
1. **自製 ASCII 渲染器** - 拒絕，因為複雜度高且無必要
2. **第三方 ASCII 函式庫** - 拒絕，因為 Three.js 原生支援更佳

---

## 2. TrackballControls 研究

### Decision
使用 Three.js 內建的 `TrackballControls` 提供相機互動控制。

### Rationale
- 支援旋轉、縮放、平移三種操作模式
- 與 AsciiEffect 的 domElement 相容
- 官方維護，穩定可靠

### API 詳細說明

#### Constructor
```javascript
new TrackballControls(camera, domElement)
```

**Parameters**:
| 參數 | 類型 | 說明 |
|------|------|------|
| `camera` | Camera | Three.js 相機實例 |
| `domElement` | HTMLElement | 接收滑鼠事件的 DOM 元素 |

#### Key Properties
| 屬性 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| `rotateSpeed` | number | 1.0 | 旋轉速度 |
| `zoomSpeed` | number | 1.2 | 縮放速度 |
| `panSpeed` | number | 0.3 | 平移速度 |

#### Methods
| 方法 | 說明 |
|------|------|
| `update()` | 更新控制器狀態（每幀呼叫） |

### 實作範例
```javascript
import { TrackballControls } from 'three/addons/controls/TrackballControls.js';

// 注意：使用 effect.domElement 而非 renderer.domElement
const controls = new TrackballControls(camera, effect.domElement);

// 動畫迴圈中更新
function animate() {
    controls.update();
    effect.render(scene, camera);
}
```

### Alternatives Considered
1. **OrbitControls** - 拒絕，因為官方範例使用 TrackballControls
2. **手動實作** - 拒絕，因為不必要的複雜度

---

## 3. CDN 與 Import Map 設定

### Decision
使用 jsDelivr CDN 載入 Three.js r170.0 及其 addons。

### Rationale
- jsDelivr 提供穩定的 npm 套件 CDN 服務
- 支援 ES6 模組和 importmap
- 與專案現有範例一致

### Import Map 配置
```html
<script type="importmap">
{
    "imports": {
        "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
        "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
    }
}
</script>
```

### 模組導入
```javascript
import * as THREE from 'three';
import { AsciiEffect } from 'three/addons/effects/AsciiEffect.js';
import { TrackballControls } from 'three/addons/controls/TrackballControls.js';
```

---

## 4. 動畫與渲染最佳實踐

### Decision
使用 requestAnimationFrame 配合時間戳記進行動畫。

### Rationale
- 瀏覽器原生支援，效能最佳
- 自動同步螢幕更新頻率
- 支援暫停時自動停止渲染

### 動畫時間計算
```javascript
const start = Date.now();

function animate() {
    const timer = Date.now() - start;
    
    // 彈跳動畫（使用正弦函數）
    sphere.position.y = Math.abs(Math.sin(timer * 0.002)) * 150;
    
    // 旋轉動畫
    sphere.rotation.x = timer * 0.0003;
    sphere.rotation.z = timer * 0.0002;
}
```

---

## 5. 視窗大小調整處理

### Decision
監聽 window resize 事件，同步更新相機、渲染器和效果器。

### 實作範例
```javascript
window.addEventListener('resize', onWindowResize);

function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    
    renderer.setSize(window.innerWidth, window.innerHeight);
    effect.setSize(window.innerWidth, window.innerHeight);
}
```

---

## Summary

所有技術問題已解決，無待澄清事項：

| 項目 | 狀態 | 決策 |
|------|------|------|
| AsciiEffect API | ✅ 完成 | 使用官方 addon，字元集 ` .:-+*=%@#`，invert: true |
| TrackballControls | ✅ 完成 | 綁定到 effect.domElement |
| CDN 載入 | ✅ 完成 | jsDelivr + Three.js r170.0 |
| 動畫實作 | ✅ 完成 | requestAnimationFrame + Date.now() |
| 響應式設計 | ✅ 完成 | resize 事件監聽 |
