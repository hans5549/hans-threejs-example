# API Contracts: WebGL 文字描邊效果

**Feature Branch**: `018-text-stroke`  
**Created**: 2025-12-04

## Module Interface

### generateStrokeText

建立具有描邊效果的 3D 文字群組。

```typescript
/**
 * 產生帶有描邊效果的 3D 文字
 * @param font - Three.js Font 物件
 * @param material - 材質配置物件
 * @param message - 要顯示的文字內容
 * @param size - 文字大小（單位：場景單位）
 * @param direction - 文字排列方向，預設 'ltr'
 * @returns THREE.Group 包含填充和描邊網格
 */
function generateStrokeText(
  font: Font,
  material: MaterialConfig,
  message: string,
  size: number,
  direction?: 'ltr' | 'rtl' | 'tb'
): THREE.Group;
```

### MaterialConfig

材質配置介面定義。

```typescript
interface MaterialConfig {
  /** 實心材質，用於描邊線條 */
  dark: THREE.MeshBasicMaterial;
  
  /** 半透明材質，用於填充面 */
  lite: THREE.MeshBasicMaterial;
  
  /** 基礎顏色，用於產生描邊樣式 */
  color: THREE.Color;
}
```

## External Dependencies

### Three.js Core

| 模組 | 用途 |
|------|------|
| THREE.Scene | 3D 場景容器 |
| THREE.PerspectiveCamera | 透視相機 |
| THREE.WebGLRenderer | WebGL 渲染器 |
| THREE.Group | 物件群組 |
| THREE.Mesh | 網格物件 |
| THREE.ShapeGeometry | 形狀幾何體 |
| THREE.MeshBasicMaterial | 基礎材質 |
| THREE.Color | 顏色物件 |
| THREE.FileLoader | 檔案載入器 |

### Three.js Addons

| 模組 | 路徑 | 用途 |
|------|------|------|
| Font | three/addons/loaders/FontLoader.js | 字體物件 |
| SVGLoader | three/addons/loaders/SVGLoader.js | 描邊生成 |
| OrbitControls | three/addons/controls/OrbitControls.js | 軌道控制器 |
| unzipSync, strFromU8 | three/addons/libs/fflate.module.js | zip 解壓縮 |

## Event Contracts

### OrbitControls Events

```typescript
// 視角變更時觸發渲染
controls.addEventListener('change', () => {
  renderer.render(scene, camera);
});
```

### Window Events

```typescript
// 視窗大小變更時調整渲染
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.render(scene, camera);
});
```

## File Format Contracts

### typeface.json

字體檔案格式（typeface.js 格式）。

```typescript
interface TypefaceJson {
  glyphs: {
    [char: string]: {
      ha: number;           // 水平前進量
      x_min: number;        // 字形左邊界
      x_max: number;        // 字形右邊界
      o: string;            // 輪廓命令字串
    };
  };
  familyName: string;       // 字體家族名稱
  ascender: number;         // 上升高度
  descender: number;        // 下降高度
  underlinePosition: number;
  underlineThickness: number;
  boundingBox: {
    yMin: number;
    xMin: number;
    yMax: number;
    xMax: number;
  };
  resolution: number;
  original_font_information: object;
}
```

## Error Handling Contracts

### 字體載入錯誤

```typescript
new THREE.FileLoader()
  .setResponseType('arraybuffer')
  .load(
    fontPath,
    onLoad,    // 成功回調
    onProgress, // 進度回調（可選）
    onError    // 錯誤回調
  );

function onError(error: ErrorEvent): void {
  console.error('無法載入字體檔案:', error);
  // 顯示錯誤訊息給使用者
}
```

### WebGL 支援檢測

```typescript
function checkWebGLSupport(): boolean {
  try {
    const canvas = document.createElement('canvas');
    return !!(
      window.WebGLRenderingContext &&
      (canvas.getContext('webgl') || canvas.getContext('experimental-webgl'))
    );
  } catch (e) {
    return false;
  }
}
```
