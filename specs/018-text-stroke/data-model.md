# Data Model: WebGL 文字描邊效果

**Feature Branch**: `018-text-stroke`  
**Created**: 2025-12-04  
**Status**: Complete

## Core Entities

### StrokeText (THREE.Group)

描邊文字的複合物件，包含填充層和描邊層。

```
StrokeText
├── fillMesh: THREE.Mesh          # 半透明填充文字
│   ├── geometry: ShapeGeometry   # 從 Font.generateShapes 產生
│   └── material: MeshBasicMaterial (transparent)
│
└── strokeMeshes: THREE.Mesh[]    # 描邊線條（每個形狀一個）
    ├── geometry: BufferGeometry  # 從 SVGLoader.pointsToStroke 產生
    └── material: MeshBasicMaterial (solid)
```

### Font

Three.js 字體物件，從 typeface.json 解析而來。

| 屬性/方法 | 類型 | 描述 |
|----------|------|------|
| data | Object | 字體定義資料 |
| generateShapes(text, size, direction) | Shape[] | 產生文字形狀陣列 |

### Shape

Three.js 形狀物件，表示單一字元的輪廓。

| 屬性 | 類型 | 描述 |
|------|------|------|
| curves | Curve[] | 形狀的曲線路徑 |
| holes | Path[] | 內部孔洞（如字母 O 的內圈） |
| getPoints() | Vector2[] | 取得形狀的點陣列 |

### Material Configuration

材質配置物件，用於統一管理填充和描邊材質。

```typescript
interface MaterialConfig {
  dark: THREE.MeshBasicMaterial;  // 實心描邊材質
  lite: THREE.MeshBasicMaterial;  // 半透明填充材質
  color: THREE.Color;             // 基礎顏色
}
```

### Text Direction

文字排列方向列舉。

| 值 | 描述 | 適用語言 |
|----|------|----------|
| 'ltr' | 左到右 | 英文、拉丁文、中文（現代） |
| 'rtl' | 右到左 | 希伯來文、阿拉伯文 |
| 'tb' | 上到下 | 傳統中文、日文 |

## Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         初始化階段                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. 載入字體檔案 (zip)                                           │
│     └─> FileLoader.load() ─> arraybuffer                        │
│                                                                  │
│  2. 解壓縮並解析                                                 │
│     └─> unzipSync() ─> strFromU8() ─> JSON.parse()              │
│                                                                  │
│  3. 建立 Font 物件                                               │
│     └─> new Font(json)                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      文字生成階段                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  4. 產生文字形狀                                                 │
│     └─> font.generateShapes(text, size, direction)              │
│         └─> Shape[] (含 holes)                                  │
│                                                                  │
│  5. 建立填充幾何體                                               │
│     └─> new ShapeGeometry(shapes)                               │
│         └─> computeBoundingBox() ─> translate(xMid)             │
│                                                                  │
│  6. 建立填充網格                                                 │
│     └─> new Mesh(geometry, matLite)                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      描邊生成階段                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  7. 提取孔洞形狀                                                 │
│     └─> for each shape: push holes to shapes[]                  │
│                                                                  │
│  8. 建立描邊樣式                                                 │
│     └─> SVGLoader.getStrokeStyle(width, color)                  │
│                                                                  │
│  9. 為每個形狀建立描邊                                           │
│     └─> for each shape:                                         │
│         ├─> shape.getPoints()                                   │
│         ├─> SVGLoader.pointsToStroke(points, style)             │
│         └─> new Mesh(geometry, matDark)                         │
│                                                                  │
│ 10. 組合成 Group                                                 │
│     └─> group.add(fillMesh, ...strokeMeshes)                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       渲染階段                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ 11. 定位文字群組                                                 │
│     └─> group.position.set(x, y, z)                             │
│                                                                  │
│ 12. 加入場景                                                     │
│     └─> scene.add(group)                                        │
│                                                                  │
│ 13. 渲染                                                         │
│     └─> renderer.render(scene, camera)                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## State Management

### 全域狀態

| 變數 | 類型 | 描述 |
|------|------|------|
| camera | THREE.PerspectiveCamera | 透視相機 |
| scene | THREE.Scene | 3D 場景 |
| renderer | THREE.WebGLRenderer | WebGL 渲染器 |
| controls | OrbitControls | 軌道控制器 |

### 生命週期

1. **初始化** (init)
   - 建立相機、場景、渲染器
   - 載入字體檔案
   - 設定控制器和事件監聽

2. **字體載入回調**
   - 解析字體資料
   - 建立材質配置
   - 生成三組描邊文字（英文、希伯來文、中文）
   - 定位並加入場景
   - 首次渲染

3. **互動更新**
   - OrbitControls 變更時觸發渲染
   - 視窗大小變更時更新相機和渲染器

## Validation Rules

### 字體檔案
- 必須是有效的 typeface.json 格式
- 必須包含請求的字元字形資料

### 文字參數
- size > 0
- direction ∈ {'ltr', 'rtl', 'tb'}
- message 非空字串

### 描邊樣式
- strokeWidth > 0
- color 為有效的 CSS 顏色字串
