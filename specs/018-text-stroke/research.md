# Research: WebGL 文字描邊效果

**Feature Branch**: `018-text-stroke`  
**Created**: 2025-12-04  
**Status**: Complete

## Research Tasks

### 1. Font 類別與 generateShapes 方法

**Decision**: 使用 Three.js FontLoader 和 Font 類別載入 typeface.json 格式字體，並使用 `generateShapes(text, size, direction)` 方法產生文字形狀。

**Rationale**:
- Three.js 原生支援 typeface.json 格式
- `generateShapes` 方法支援 `direction` 參數（'ltr', 'rtl', 'tb'），可控制文字排列方向
- 返回 `Shape[]` 物件，每個形狀包含外輪廓和內部孔洞（holes）

**Alternatives considered**:
- TextGeometry：僅支援 3D 擠出文字，無法分離填充和描邊
- 手動解析 TTF/OTF：過於複雜，Three.js 已有成熟方案

### 2. SVGLoader.pointsToStroke 描邊技術

**Decision**: 使用 SVGLoader 的靜態方法 `getStrokeStyle` 和 `pointsToStroke` 將形狀輪廓轉換為描邊幾何體。

**Rationale**:
- `SVGLoader.getStrokeStyle(strokeWidth, color)` 建立描邊樣式配置
- `SVGLoader.pointsToStroke(points, style)` 將點陣列轉換為 BufferGeometry
- 此方法可精確控制描邊寬度和外觀

**Implementation pattern**:
```javascript
const style = SVGLoader.getStrokeStyle(5, color.getStyle());
const points = shape.getPoints();
const geometry = SVGLoader.pointsToStroke(points, style);
```

### 3. 文字方向支援（LTR、RTL、TB）

**Decision**: 使用 Font.generateShapes 的第三個參數 `direction` 控制文字排列方向。

**Rationale**:
- Three.js r166+ 版本的 FontLoader 已原生支援 `direction` 參數
- 'ltr'：左到右（英文、拉丁文）
- 'rtl'：右到左（希伯來文、阿拉伯文）
- 'tb'：上到下（傳統中文、日文）

**Alternatives considered**:
- 手動旋轉/排列：複雜且難以維護
- CSS text-direction：不適用於 WebGL 渲染

### 4. 孔洞處理（Hole Shapes）

**Decision**: 從 Shape 物件中提取 holes 屬性，將孔洞也加入描邊路徑。

**Rationale**:
- 字母如 O、B、D 等包含內部孔洞
- ShapeGeometry 會自動處理孔洞的填充
- 描邊需要手動遍歷 `shape.holes` 並加入描邊路徑

**Implementation pattern**:
```javascript
const holeShapes = [];
for (const shape of shapes) {
  if (shape.holes?.length > 0) {
    holeShapes.push(...shape.holes);
  }
}
shapes.push(...holeShapes);
```

### 5. 字體檔案載入策略

**Decision**: 使用 fflate 函式庫解壓縮 zip 格式的字體檔案，減少網路傳輸量。

**Rationale**:
- typeface.json 字體檔案通常較大（數百 KB 到數 MB）
- zip 壓縮可顯著減少傳輸大小
- Three.js 已內建 fflate 函式庫（`three/addons/libs/fflate.module.js`）

**Implementation pattern**:
```javascript
import { unzipSync, strFromU8 } from 'three/addons/libs/fflate.module.js';

new THREE.FileLoader()
  .setResponseType('arraybuffer')
  .load('font.typeface.json.zip', (data) => {
    const zip = unzipSync(new Uint8Array(data));
    const json = strFromU8(zip['font.typeface.json']);
    const font = new Font(JSON.parse(json));
  });
```

### 6. 材質配置策略

**Decision**: 使用兩種 MeshBasicMaterial：半透明填充和實心描邊。

**Rationale**:
- 填充層使用半透明材質，讓使用者能看到描邊的層次感
- 描邊層使用實心材質，確保線條清晰可見
- DoubleSide 確保從任何角度都能看到文字

**Configuration**:
```javascript
const matLite = new THREE.MeshBasicMaterial({
  color: 0x006699,
  transparent: true,
  opacity: 0.4,
  side: THREE.DoubleSide
});

const matDark = new THREE.MeshBasicMaterial({
  color: 0x006699,
  side: THREE.DoubleSide
});
```

### 7. 文字置中對齊

**Decision**: 使用 geometry.computeBoundingBox() 計算邊界框，然後平移幾何體使其置中。

**Rationale**:
- 預設情況下文字從原點開始繪製
- 需要計算 bounding box 的 x 軸中點
- 將幾何體平移 `-xMid` 使其水平置中

**Implementation pattern**:
```javascript
geometry.computeBoundingBox();
const xMid = -0.5 * (geometry.boundingBox.max.x - geometry.boundingBox.min.x);
geometry.translate(xMid, 0, 0);
```

## Technology Stack Confirmation

| 元件 | 技術選擇 | 版本 |
|------|----------|------|
| 3D 引擎 | Three.js | 0.170.0 |
| 字體載入 | Font (FontLoader) | Three.js 內建 |
| 描邊生成 | SVGLoader | Three.js 內建 |
| 解壓縮 | fflate | Three.js 內建 |
| 互動控制 | OrbitControls | Three.js 內建 |

## Dependencies

### 外部相依

- Three.js CDN（https://cdn.jsdelivr.net/npm/three@0.170.0/）
- 字體檔案：MPLUSRounded1c-Regular.typeface.json.zip（需從 Three.js 範例下載）

### 內部相依

無（獨立範例）

## Risks and Mitigations

| 風險 | 影響 | 緩解策略 |
|------|------|----------|
| 字體檔案過大 | 載入時間長 | 使用 zip 壓縮、考慮子集化 |
| 某些字體缺少字元 | 文字無法正確顯示 | 選擇支援多語言的字體 |
| 瀏覽器不支援 WebGL | 無法顯示 | 添加 WebGL 檢測和提示 |
| direction 參數不支援 | RTL/TB 文字排列錯誤 | 確保使用 Three.js r166+ 版本 |

## Resolved Clarifications

所有技術問題均已透過研究解決，無需額外澄清。
