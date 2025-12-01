# Implementation Plan: WebGL Raycaster BVH 高效能光線投射

**Branch**: `8-raycaster-bvh` | **Date**: 2025-12-01 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/8-raycaster-bvh/spec.md`

## Summary

建立一個展示 BVH (Bounding Volume Hierarchy) 加速光線投射效能的 Three.js 範例。系統載入 3D 模型（Stanford Bunny FBX）後，在模型周圍球體表面隨機投射多條光線，視覺化光線路徑和相交點，並提供 BVH 開關讓使用者比較效能差異（預期從 60 FPS 降至 10-20 FPS）。技術方案採用單一 HTML 檔案結構，整合 three-mesh-bvh 函式庫，實現高效能空間查詢和即時效能監控。

## Technical Context

**Language/Version**: JavaScript ES6+ (ES Modules)  
**Primary Dependencies**: 
- Three.js r160+ (via CDN - unpkg/jsDelivr)
- three-mesh-bvh v0.7.3+ (BVH 加速結構)
- FBXLoader (載入 FBX 模型)
- OrbitControls (場景互動)
- Stats.js (FPS 監控)
- lil-gui (控制介面)

**Storage**: N/A (純前端展示，無持久化需求)  
**Testing**: 手動瀏覽器測試 + 效能比較驗證  
**Target Platform**: 現代瀏覽器 (Chrome, Firefox, Safari, Edge) - WebGL 2.0 支援  
**Project Type**: Single HTML file (獨立展示範例)  
**Performance Goals**: 
- 啟用 BVH: 60+ FPS (100 條光線)
- 停用 BVH: 10-20 FPS (70% 效能下降)
- 控制回應: <500ms
- 載入時間: <60 秒 (含模型下載)

**Constraints**: 
- FPS < 30 觸發動態警告
- 載入逾時: 60 秒
- 視覺化延遲: <100ms (BVH 切換)
- 無光線數量硬上限（動態調整）

**Scale/Scope**: 
- 預設 100 條光線（可調整）
- Stanford Bunny 模型 (~70k 三角形)
- 即時 BVH 建構和查詢
- 200 個實例化物件 (光線起點 + 相交點)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 原則 | 狀態 | 說明 |
|------|------|------|
| 簡單性 (Simplicity) | ✅ Pass | 單一 HTML 檔案，無複雜建置流程 |
| 獨立性 (Self-contained) | ✅ Pass | 僅依賴 CDN 載入的函式庫，可獨立執行 |
| 可測試性 (Testable) | ✅ Pass | 開啟瀏覽器即可驗證功能和效能差異 |
| 文件完整性 | ✅ Pass | 規格已完成釐清，所有決策點已明確 |
| 效能可觀察性 | ✅ Pass | Stats.js 提供即時 FPS 監控和警告機制 |

**Gate Result**: ✅ PASS - 可進入 Phase 0

## Project Structure

### Documentation (this feature)

```text
specs/8-raycaster-bvh/
├── spec.md              # 功能規格 (已完成，含釐清記錄)
├── plan.md              # 本檔案 - 實作計畫
├── research.md          # Phase 0 輸出 - 技術研究
├── data-model.md        # Phase 1 輸出 - 資料模型
├── quickstart.md        # Phase 1 輸出 - 快速開始指南
├── contracts/           # Phase 1 輸出 - API 契約
│   └── component-interface.md
└── checklists/
    └── requirements.md  # 需求檢查清單 (已完成)
```

### Source Code (repository root)

```text
examples/
└── webgl-raycaster-bvh/
    └── index.html       # 單一 HTML 檔案 (內嵌 CSS + JS)

models/
└── fbx/
    └── stanford-bunny.fbx  # 3D 模型檔案 (從 Three.js 範例路徑)
```

**Structure Decision**: 採用單一 HTML 檔案結構，與 Three.js 官方範例一致。模型檔案使用 Three.js 官方範例的路徑策略，確保可靠性和可維護性。

## Complexity Tracking

> 無違規需要記錄 - 本專案採用最簡單的單檔案結構，符合教育展示性質

---

## Phase 0: Research

### 研究任務清單

#### 1. BVH 整合與 API 研究
- 研究 three-mesh-bvh 的 computeBoundsTree API 和配置選項
- 確認如何擴展 BufferGeometry.prototype 來啟用 BVH
- 研究 acceleratedRaycast 的整合方式和效能影響
- 確認 MeshBVHHelper 的視覺化配置和更新機制

#### 2. FBX 模型載入策略
- 確認 FBXLoader 的非同步載入模式和錯誤處理
- 研究載入進度事件 (onProgress) 的回呼機制
- 確認 60 秒逾時的實作方式 (Promise.race 或 AbortController)
- 研究模型載入失敗的錯誤類型和處理策略

#### 3. 光線投射演算法
- 研究球體表面均勻取樣演算法 (spherical coordinates)
- 確認光線原點和方向的計算方式
- 研究 Raycaster 的 far 參數設定和相交檢測
- 確認如何區分有/無相交的光線視覺化

#### 4. 實例化渲染優化
- 研究 InstancedMesh 的矩陣更新機制 (setMatrixAt)
- 確認 DynamicDrawUsage 的效能影響
- 研究 LineSegments 的頂點屬性動態更新
- 確認 setDrawRange 的使用場景和限制

#### 5. 效能監控與警告機制
- 確認 Stats.js 的 FPS 讀取 API
- 研究如何實作 FPS < 30 的動態警告邏輯
- 確認 requestAnimationFrame 的幀時間計算
- 研究記憶體使用量的監控方式 (performance.memory)

#### 6. GUI 控制介面
- 研究 lil-gui 的初始化和控制項類型
- 確認如何綁定 BVH 開關的回呼函式
- 研究滑桿 (slider) 的數值範圍和步進設定
- 確認 GUI 更新時的即時回應機制

### 預期研究結果結構

詳見 [research.md](./research.md) - 將包含：
- 決策記錄 (Decision)：選擇的技術方案和原因
- 替代方案 (Alternatives)：評估過但未採用的方案
- 實作考量 (Implementation Notes)：關鍵細節和注意事項
- 程式碼範例 (Code Examples)：關鍵 API 的使用示範

---

## Phase 1: Design & Contracts

### 1.1 資料模型設計

詳見 [data-model.md](./data-model.md) - 將定義：

**核心實體**:
- **Scene 場景狀態**：模型、BVH、攝影機、渲染器的生命週期
- **Ray 光線**：位置、方向、最大距離、相交結果
- **BVHTree BVH 樹**：節點結構、包圍盒、深度資訊
- **PerformanceMetrics 效能指標**：FPS、幀時間、記憶體、警告狀態
- **LoadingState 載入狀態**：進度百分比、錯誤訊息、逾時狀態

**狀態轉換**:
- 載入流程：IDLE → LOADING → LOADED / ERROR
- BVH 狀態：DISABLED → BUILDING → ENABLED → DISABLED

### 1.2 API 契約設計

詳見 [contracts/component-interface.md](./contracts/component-interface.md) - 將定義：

**核心函式介面**:
```javascript
// 場景初始化
init(): void

// 模型載入（含逾時和進度）
loadModel(path: string, timeout: number): Promise<Object3D>

// BVH 操作
buildBVH(geometry: BufferGeometry): void
disposeBVH(geometry: BufferGeometry): void
toggleBVH(enabled: boolean): void

// 光線投射
generateRandomRays(count: number): Ray[]
castRays(rays: Ray[]): IntersectionResult[]
updateRayVisualization(rays: Ray[], results: IntersectionResult[]): void

// 效能監控
updatePerformanceMetrics(): PerformanceMetrics
checkPerformanceWarning(fps: number): boolean

// GUI 控制
createGUI(config: GUIConfig): GUI
updateGUIState(state: AppState): void
```

**事件回呼**:
- onLoadProgress(percent: number): void
- onLoadError(error: Error): void
- onBVHToggle(enabled: boolean): void
- onRayCountChange(count: number): void
- onPerformanceWarning(fps: number): void

### 1.3 快速開始指南

詳見 [quickstart.md](./quickstart.md) - 將包含：
- 開啟方式：直接用瀏覽器開啟 index.html
- 互動操作：滑鼠控制、GUI 調整、效能觀察
- 驗收測試：BVH 效能比較、視覺化驗證
- 故障排除：常見問題和解決方案

---

## Phase 2: Task Planning

**注意**: Phase 2 的 tasks.md 由 `/speckit.tasks` 指令產生，不在本計畫檔案中建立。

任務分解將基於以下結構：
1. **T1 - 專案架構設定** (P1)
   - 建立 HTML 檔案結構
   - 配置 import maps 載入 Three.js 和相依套件
   - 設定基本場景、攝影機、渲染器

2. **T2 - 模型載入系統** (P1)
   - 實作 FBXLoader 整合
   - 實作載入進度指示器
   - 實作 60 秒逾時機制
   - 實作錯誤處理和訊息顯示

3. **T3 - BVH 整合** (P1)
   - 整合 three-mesh-bvh 函式庫
   - 實作 BVH 建構和銷毀
   - 實作 acceleratedRaycast 切換
   - 實作 MeshBVHHelper 視覺化

4. **T4 - 光線投射系統** (P2)
   - 實作球體表面隨機取樣
   - 實作光線批次投射
   - 實作相交檢測和結果記錄
   - 實作光線和相交點視覺化 (InstancedMesh + LineSegments)

5. **T5 - 效能監控** (P2)
   - 整合 Stats.js
   - 實作 FPS 監控和記錄
   - 實作 FPS < 30 動態警告
   - 實作效能指標顯示

6. **T6 - GUI 控制介面** (P3)
   - 整合 lil-gui
   - 實作 BVH 開關控制
   - 實作 BVH 視覺化切換
   - 實作光線數量滑桿

7. **T7 - 場景互動** (P3)
   - 整合 OrbitControls
   - 實作模型自動旋轉
   - 實作視窗大小調整處理

8. **T8 - 測試與優化** (P3)
   - 驗收測試執行
   - 效能基準測試
   - 跨瀏覽器相容性測試
   - 文件完善和程式碼註解

---

## 關鍵決策記錄

### 決策 1: 模型來源策略
- **決策**: 使用 Three.js 官方範例的模型路徑 (`models/fbx/stanford-bunny.fbx`)
- **理由**: 確保模型可用性，降低維護成本，與官方範例一致
- **替代方案**: 本地專案目錄、CDN 載入、多來源備援
- **影響**: 需確保專案目錄結構包含 models/ 資料夾

### 決策 2: 載入體驗設計
- **決策**: 顯示空場景（背景、光照）加載入進度指示器
- **理由**: 提供即時回饋，保持簡潔，避免複雜實作
- **替代方案**: 靜態佔位畫面、低解析度預覽、線框骨架
- **影響**: 需實作進度百分比計算和顯示邏輯

### 決策 3: 光線數量策略
- **決策**: 無硬上限，FPS < 30 時動態顯示警告
- **理由**: 給予使用者最大彈性，根據實際效能動態調整
- **替代方案**: 固定上限（300、500、1000）
- **影響**: 需實作 FPS 監控和警告邏輯，使用者可能設定過高值

### 決策 4: 載入失敗處理
- **決策**: 60 秒逾時，顯示錯誤訊息（需重新整理）
- **理由**: 給予充足載入時間，失敗後手動控制恢復
- **替代方案**: 自動重試、較短逾時、無逾時限制
- **影響**: 網路不穩時使用者體驗較差，但實作簡單

### 決策 5: 光線分布策略
- **決策**: 球體表面隨機取樣，指向模型中心
- **理由**: 均勻覆蓋各方向，視覺效果最佳，符合原始範例
- **替代方案**: 攝影機方向、完全隨機、預定義位置
- **影響**: 需實作球面座標轉換演算法

---

## 驗收標準對應

基於規格 [spec.md](./spec.md) 中的成功標準：

| 成功標準 ID | 驗收方式 | 對應任務 |
|------------|---------|---------|
| SC-001 | 開啟範例，啟用 BVH，觀察 Stats.js 顯示 FPS ≥ 60 | T3, T5 |
| SC-002 | 切換停用 BVH，觀察 FPS 下降至 18 以下（≥70% 下降） | T3, T5 |
| SC-003 | 使用 GUI 切換 BVH，在 3 秒內看到 FPS 變化 | T6, T5 |
| SC-004 | 預設 100 條光線情況下，畫面流暢無卡頓 | T4, T5 |
| SC-005 | 模型載入時，2 秒內顯示進度指示器 | T2 |
| SC-006 | 調整光線數量滑桿，0.5 秒內更新視覺化 | T6, T4 |
| SC-007 | 切換 BVH 視覺化，包圍盒即時顯示/隱藏 | T3, T6 |
| SC-008 | 滑鼠拖曳旋轉場景，無明顯延遲 | T7 |

---

## 風險緩解計畫

### 風險 1: BVH 效能提升不明顯
- **緩解**: Phase 0 研究階段確認 three-mesh-bvh 在 Stanford Bunny 模型上的實際效能數據
- **應變**: 如效果不佳，考慮使用更複雜的模型或增加光線數量

### 風險 2: 模型載入失敗率高
- **緩解**: 在 Phase 1 設計階段確認模型檔案的可存取性和大小
- **應變**: 準備備用 CDN 路徑或使用較小的替代模型

### 風險 3: 低階裝置效能過差
- **緩解**: 實作 FPS < 30 警告機制，引導使用者降低光線數量
- **應變**: 提供預設配置建議（如行動裝置使用 50 條光線）

### 風險 4: 記憶體洩漏
- **緩解**: 在 Phase 1 設計清理資源的明確流程
- **應變**: 實作記憶體監控和定期垃圾回收提示

---

## 下一步

1. ✅ Phase 0 完成標準：research.md 建立，所有研究任務完成
2. ✅ Phase 1 完成標準：data-model.md, contracts/, quickstart.md 建立
3. ⏭️ Phase 2 執行：使用 `/speckit.tasks` 指令產生 tasks.md

**當前狀態**: Phase 0 就緒，可開始技術研究
