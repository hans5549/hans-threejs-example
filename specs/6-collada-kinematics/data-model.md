# Data Model: COLLADA Kinematics 機器人手臂動畫

**Date**: 2025-11-28 | **Spec**: [spec.md](./spec.md) | **Research**: [research.md](./research.md)

## 概述

本文件定義 COLLADA Kinematics 機器人展示應用的資料模型，包含機器人目錄、運動學狀態、UI 狀態等核心實體。

---

## 1. 核心實體

### 1.1 RobotCatalog（機器人目錄）

靜態資料，定義可用的機器人模型清單。

```typescript
interface RobotCatalog {
    id: string;              // 唯一識別碼，例如 "abb", "kuka", "ur6"
    name: string;            // 顯示名稱，例如 "ABB IRB 52"
    manufacturer: string;    // 製造商
    modelFile: string;       // .dae 檔案路徑
    description: string;     // 簡短描述
    defaultScale: number;    // 預設縮放比例
    defaultPosition: {       // 預設位置
        x: number;
        y: number;
        z: number;
    };
}
```

**資料實例**:

| id | name | manufacturer | modelFile | defaultScale |
|----|------|--------------|-----------|--------------|
| abb | ABB IRB 52 | ABB | models/abb_irb52_7_120.dae | 1.0 |
| kuka | KUKA KR5-R650 | KUKA | models/kuka-kr5-r650.dae | 1.0 |
| ur6 | Universal Robots UR6 | Universal Robots | models/universalrobots-ur6-85-5-a.dae | 1.0 |

---

### 1.2 KinematicsJoint（運動學關節）

從 COLLADA 檔案解析的關節資料。

```typescript
interface KinematicsJoint {
    name: string;            // 關節名稱
    static: boolean;         // 是否為靜態關節（不可動）
    limits: {
        min: number;         // 最小角度/位置
        max: number;         // 最大角度/位置
    };
    currentValue: number;    // 當前值
}
```

**驗證規則**:
- `currentValue` 必須在 `limits.min` 和 `limits.max` 之間
- 若 `static === true`，則忽略 limits 和 currentValue

---

### 1.3 RobotState（機器人狀態）

執行時期的機器人實例狀態。

```typescript
interface RobotState {
    currentRobotId: string;          // 當前載入的機器人 ID
    scene: THREE.Group | null;       // Three.js 場景物件
    kinematics: ColladaKinematics | null;  // 運動學物件
    joints: Map<string, KinematicsJoint>;  // 關節對映
    isLoading: boolean;              // 載入狀態
    loadProgress: number;            // 載入進度 0-100
    hasKinematics: boolean;          // 是否有運動學資料
    errorMessage: string | null;     // 錯誤訊息
}
```

**狀態轉換**:

```
IDLE → LOADING → LOADED/ERROR
         ↑______________|
         (切換模型時)
```

---

### 1.4 AnimationState（動畫狀態）

TWEEN 動畫的狀態管理。

```typescript
interface AnimationState {
    isAnimating: boolean;            // 是否正在播放
    currentTween: TWEEN.Tween | null; // 當前動畫實例
    duration: number;                // 動畫時長 (ms)
    targetValues: Record<string, number>;  // 目標關節值
}
```

**驗證規則**:
- `duration` 必須在 1000-5000ms 之間（隨機生成）
- `targetValues` 的每個值必須在對應關節的 limits 範圍內

---

### 1.5 CameraState（攝影機狀態）

攝影機環繞動畫狀態。

```typescript
interface CameraState {
    theta: number;          // 水平角度 (弧度)
    radius: number;         // 距離中心點的半徑
    targetY: number;        // 觀察目標 Y 座標
    autoRotate: boolean;    // 是否自動旋轉
    rotationSpeed: number;  // 旋轉速度
}
```

**預設值**:
- `radius`: 3
- `targetY`: 1.5
- `autoRotate`: true
- `rotationSpeed`: 0.001

---

### 1.6 UIState（介面狀態）

使用者介面狀態。

```typescript
interface UIState {
    selectedRobotId: string;         // 選擇器當前選中的機器人
    showLoadingOverlay: boolean;     // 是否顯示載入遮罩
    loadingText: string;             // 載入提示文字
    showStats: boolean;              // 是否顯示 FPS 面板
    showInfo: boolean;               // 是否顯示機器人資訊
}
```

---

## 2. 關係圖

```
┌─────────────────┐
│  RobotCatalog   │ (靜態設定)
│  [Array<Robot>] │
└────────┬────────┘
         │ 選擇
         ▼
┌─────────────────┐      ┌─────────────────┐
│   RobotState    │ ◄──► │ AnimationState  │
│  (執行時狀態)    │      │  (TWEEN 管理)   │
└────────┬────────┘      └─────────────────┘
         │
         │ 包含
         ▼
┌─────────────────┐
│ KinematicsJoint │
│   [Map<Joint>]  │
└─────────────────┘

┌─────────────────┐      ┌─────────────────┐
│  CameraState    │      │    UIState      │
│  (環繞動畫)      │      │  (介面控制)     │
└─────────────────┘      └─────────────────┘
```

---

## 3. 應用程式狀態

整合所有狀態的全域應用程式狀態：

```typescript
interface AppState {
    // 靜態資料
    catalog: RobotCatalog[];
    
    // 執行時狀態
    robot: RobotState;
    animation: AnimationState;
    camera: CameraState;
    ui: UIState;
    
    // Three.js 物件參照
    scene: THREE.Scene;
    renderer: THREE.WebGLRenderer;
    camera: THREE.PerspectiveCamera;
    stats: Stats;
}
```

---

## 4. 事件流

### 4.1 模型切換流程

```
User selects robot in dropdown
         │
         ▼
┌─────────────────────────┐
│ 1. Stop current TWEEN   │
│ 2. Remove current model │
│ 3. Show loading overlay │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 4. Load new .dae file   │
│    (with progress)      │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 5. Parse kinematics     │
│ 6. Add to scene         │
│ 7. Hide loading overlay │
│ 8. Start new animation  │
└─────────────────────────┘
```

### 4.2 動畫循環流程

```
setupTween() called
         │
         ▼
┌─────────────────────────┐
│ 1. Generate random      │
│    duration (1-5s)      │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 2. Generate random      │
│    target joint values  │
│    (within limits)      │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 3. Create TWEEN         │
│    with Quadratic.Out   │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 4. On complete:         │
│    setupTween() again   │
└─────────────────────────┘
```

---

## 5. 資料驗證規則摘要

| 欄位 | 規則 |
|------|------|
| RobotCatalog.id | 非空字串，唯一 |
| RobotCatalog.modelFile | 有效的相對路徑 |
| KinematicsJoint.limits | min < max |
| KinematicsJoint.currentValue | min ≤ value ≤ max |
| AnimationState.duration | 1000 ≤ duration ≤ 5000 |
| CameraState.radius | radius > 0 |
| UIState.loadingText | 非空字串 |

---

## 6. 預設值

```javascript
const DEFAULT_ROBOT_ID = 'abb';

const DEFAULT_CAMERA_STATE = {
    theta: 0,
    radius: 3,
    targetY: 1.5,
    autoRotate: true,
    rotationSpeed: 0.001
};

const DEFAULT_ANIMATION_DURATION_RANGE = {
    min: 1000,
    max: 5000
};
```
