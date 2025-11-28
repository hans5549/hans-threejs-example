# Component Interface: COLLADA Kinematics 機器人手臂動畫

**Date**: 2025-11-28 | **Spec**: [spec.md](../spec.md) | **Data Model**: [data-model.md](../data-model.md)

## 概述

本文件定義 COLLADA Kinematics 機器人展示應用的元件介面規格，包含公開函式、事件處理和模組結構。

---

## 1. 模組結構

由於採用單一 HTML 檔案模式，所有程式碼組織為邏輯模組（命名空間）：

```javascript
// 主要模組
const App = {
    // 初始化
    init: function() {},
    
    // 子模組
    Scene: {},      // 場景管理
    Robot: {},      // 機器人載入與管理
    Animation: {},  // 動畫控制
    Camera: {},     // 攝影機控制
    UI: {}          // 使用者介面
};
```

---

## 2. 公開介面

### 2.1 App 模組

**主要進入點**

```typescript
interface App {
    /**
     * 初始化應用程式
     * @returns void
     */
    init(): void;
    
    /**
     * 主要渲染迴圈
     * @returns void
     */
    animate(): void;
}
```

---

### 2.2 Scene 模組

**場景管理**

```typescript
interface SceneModule {
    /**
     * 建立 Three.js 場景
     * @returns THREE.Scene
     */
    create(): THREE.Scene;
    
    /**
     * 建立 WebGL 渲染器
     * @returns THREE.WebGLRenderer
     */
    createRenderer(): THREE.WebGLRenderer;
    
    /**
     * 設定場景光源
     * @param scene - 目標場景
     * @returns void
     */
    setupLights(scene: THREE.Scene): void;
    
    /**
     * 設定地板網格
     * @param scene - 目標場景
     * @returns void
     */
    setupGrid(scene: THREE.Scene): void;
}
```

---

### 2.3 Robot 模組

**機器人載入與管理**

```typescript
interface RobotModule {
    /**
     * 取得機器人目錄
     * @returns RobotCatalog[]
     */
    getCatalog(): RobotCatalog[];
    
    /**
     * 載入機器人模型
     * @param robotId - 機器人識別碼
     * @param onProgress - 進度回呼 (0-100)
     * @returns Promise<LoadedRobot>
     */
    load(robotId: string, onProgress?: (percent: number) => void): Promise<LoadedRobot>;
    
    /**
     * 卸載當前機器人
     * @returns void
     */
    unload(): void;
    
    /**
     * 切換機器人模型
     * @param robotId - 目標機器人識別碼
     * @returns Promise<void>
     */
    switchTo(robotId: string): Promise<void>;
    
    /**
     * 取得當前機器人資訊
     * @returns RobotInfo | null
     */
    getCurrentInfo(): RobotInfo | null;
}

interface LoadedRobot {
    scene: THREE.Group;
    kinematics: ColladaKinematics | null;
    joints: Map<string, KinematicsJoint>;
    hasKinematics: boolean;
}

interface RobotInfo {
    id: string;
    name: string;
    manufacturer: string;
    jointCount: number;
    hasKinematics: boolean;
}
```

---

### 2.4 Animation 模組

**動畫控制**

```typescript
interface AnimationModule {
    /**
     * 啟動關節動畫
     * @param kinematics - 運動學物件
     * @param joints - 關節資訊
     * @returns void
     */
    start(kinematics: ColladaKinematics, joints: Map<string, KinematicsJoint>): void;
    
    /**
     * 停止當前動畫
     * @returns void
     */
    stop(): void;
    
    /**
     * 更新動畫（每幀呼叫）
     * @param delta - 時間差
     * @returns void
     */
    update(delta: number): void;
    
    /**
     * 設定新的隨機目標並開始動畫
     * @returns void
     */
    setupTween(): void;
}
```

---

### 2.5 Camera 模組

**攝影機控制**

```typescript
interface CameraModule {
    /**
     * 建立透視攝影機
     * @returns THREE.PerspectiveCamera
     */
    create(): THREE.PerspectiveCamera;
    
    /**
     * 更新環繞動畫（每幀呼叫）
     * @param delta - 時間差
     * @returns void
     */
    updateOrbit(delta: number): void;
    
    /**
     * 重置攝影機位置
     * @returns void
     */
    reset(): void;
    
    /**
     * 處理視窗縮放
     * @returns void
     */
    onResize(): void;
}
```

---

### 2.6 UI 模組

**使用者介面**

```typescript
interface UIModule {
    /**
     * 初始化所有 UI 元件
     * @param catalog - 機器人目錄
     * @param onRobotChange - 選擇變更回呼
     * @returns void
     */
    init(catalog: RobotCatalog[], onRobotChange: (robotId: string) => void): void;
    
    /**
     * 顯示載入指示器
     * @param progress - 載入進度 (0-100)
     * @returns void
     */
    showLoading(progress: number): void;
    
    /**
     * 隱藏載入指示器
     * @returns void
     */
    hideLoading(): void;
    
    /**
     * 顯示錯誤訊息
     * @param message - 錯誤訊息
     * @returns void
     */
    showError(message: string): void;
    
    /**
     * 更新機器人資訊面板
     * @param info - 機器人資訊
     * @returns void
     */
    updateRobotInfo(info: RobotInfo): void;
    
    /**
     * 初始化 FPS 統計面板
     * @returns Stats
     */
    initStats(): Stats;
}
```

---

## 3. 事件處理

### 3.1 DOM 事件

| 事件 | 處理函式 | 說明 |
|------|----------|------|
| `DOMContentLoaded` | `App.init()` | 應用程式初始化 |
| `resize` | `Camera.onResize()` | 視窗縮放處理 |
| `change` (select) | `Robot.switchTo()` | 機器人選擇變更 |

### 3.2 事件流程

```
[用戶選擇機器人]
        │
        ▼
┌───────────────────┐
│ select.onchange   │
│ → UI.showLoading  │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Robot.switchTo()  │
│ → Animation.stop  │
│ → Robot.unload    │
│ → Robot.load      │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ onProgress        │
│ → UI.showLoading  │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ onLoad            │
│ → Scene.add       │
│ → Animation.start │
│ → UI.hideLoading  │
│ → UI.updateInfo   │
└───────────────────┘
```

---

## 4. HTML 結構

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>COLLADA Kinematics - Robot Arm</title>
    <style>
        /* 內嵌樣式 */
    </style>
</head>
<body>
    <!-- 載入指示器 -->
    <div id="loadingOverlay">
        <div id="loadingProgress">Loading... 0%</div>
    </div>
    
    <!-- 機器人選擇器 -->
    <select id="robotSelector">
        <option value="abb">ABB IRB 52</option>
        <option value="kuka">KUKA KR5-R650</option>
        <option value="ur6">Universal Robots UR6</option>
    </select>
    
    <!-- 機器人資訊面板 -->
    <div id="robotInfo"></div>
    
    <!-- WebGL 降級訊息 -->
    <noscript>JavaScript is required for this application.</noscript>
    
    <!-- Import Map -->
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
        // ES Module 程式碼
    </script>
</body>
</html>
```

---

## 5. CSS 樣式規格

### 5.1 基礎樣式

```css
body {
    margin: 0;
    padding: 0;
    overflow: hidden;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
```

### 5.2 載入指示器

```css
#loadingOverlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.8);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
    transition: opacity 0.3s;
}

#loadingOverlay.hidden {
    opacity: 0;
    pointer-events: none;
}

#loadingProgress {
    color: #fff;
    font-size: 24px;
    font-weight: 300;
}
```

### 5.3 機器人選擇器

```css
#robotSelector {
    position: fixed;
    top: 10px;
    right: 10px;
    padding: 8px 12px;
    font-size: 14px;
    border: none;
    border-radius: 4px;
    background: rgba(255, 255, 255, 0.9);
    cursor: pointer;
    z-index: 100;
}

#robotSelector:hover {
    background: #fff;
}
```

### 5.4 機器人資訊面板

```css
#robotInfo {
    position: fixed;
    bottom: 10px;
    left: 10px;
    padding: 12px 16px;
    background: rgba(0, 0, 0, 0.7);
    color: #fff;
    font-size: 12px;
    border-radius: 4px;
    z-index: 100;
    max-width: 250px;
}

#robotInfo .name {
    font-size: 16px;
    font-weight: 500;
    margin-bottom: 4px;
}

#robotInfo .detail {
    opacity: 0.8;
}
```

---

## 6. 錯誤處理規格

### 6.1 錯誤代碼

| 代碼 | 說明 | 處理方式 |
|------|------|----------|
| `WEBGL_NOT_SUPPORTED` | WebGL 不支援 | 顯示降級圖片和說明 |
| `MODEL_LOAD_FAILED` | 模型載入失敗 | 顯示錯誤訊息，保留當前模型 |
| `NETWORK_ERROR` | 網路錯誤 | 顯示重試選項 |
| `INVALID_KINEMATICS` | 運動學資料無效 | 顯示靜態模型 |

### 6.2 錯誤回應格式

```javascript
function handleError(code, message, originalError) {
    console.error(`[${code}] ${message}`, originalError);
    
    UI.showError({
        code: code,
        message: message,
        recoverable: code !== 'WEBGL_NOT_SUPPORTED'
    });
}
```

---

## 7. 效能規格

### 7.1 目標指標

| 指標 | 目標值 | 測量方式 |
|------|--------|----------|
| FPS | ≥ 30 | Stats.js 面板 |
| 模型載入時間 | < 5s | Console timing |
| 記憶體使用 | < 200MB | Chrome DevTools |
| 首次互動時間 | < 3s | Lighthouse |

### 7.2 最佳化策略

- 模型檔案使用 gzip 壓縮傳輸
- 使用 `requestAnimationFrame` 進行渲染迴圈
- 避免每幀建立新物件（物件池模式）
- 載入時顯示進度，避免使用者感知延遲
