# Tasks: CSS3D 週期表

**Feature**: 3-css3d-periodic-table  
**Date**: 2025-11-28  
**Generated From**: spec.md, plan.md, data-model.md, contracts/component-interface.md

---

## Summary

| Metric | Value |
|--------|-------|
| Total Tasks | 18 |
| Setup Phase | 2 tasks |
| Foundational Phase | 2 tasks |
| US1 (Table View) | 4 tasks |
| US2 (View Switching) | 4 tasks |
| US3 (Hover Effects) | 3 tasks |
| US4 (Responsive Resize) | 2 tasks |
| Polish Phase | 1 task |
| Parallel Opportunities | 6 tasks |

---

## Phase 1: Setup (專案初始化)

**Goal**: 建立檔案結構與基礎 HTML 骨架

- [X] T001 建立 `examples/css3d-periodic-table/` 目錄與 `index.html` 基礎骨架
- [X] T002 在 `index.html` 加入 CDN 依賴 (Three.js r160, TWEEN.js 21.0.0, CSS3DRenderer, TrackballControls)

**Completion Criteria**:
- [X] `index.html` 存在且可在瀏覽器開啟
- [X] CDN 腳本正確載入 (無 console 錯誤)

---

## Phase 2: Foundational (基礎建設)

**Goal**: 建立 3D 場景核心元件，為所有 User Stories 提供基礎

- [X] T003 實作 `init()` 函式：建立 PerspectiveCamera、Scene、CSS3DRenderer、container 掛載
- [X] T004 實作 `animate()` 與 `render()` 迴圈，加入 TWEEN.update()

**Completion Criteria**:
- [X] 3D 場景初始化成功
- [X] 渲染迴圈正常運作 (60fps)

**Dependencies**: Phase 1 完成

---

## Phase 3: User Story 1 - 週期表視圖 [P1]

**Story**: 作為使用者，我希望看到 118 個化學元素以標準週期表佈局呈現在 3D 空間中

**Goal**: 實作元素卡片與 Table 佈局

**Independent Test Criteria**:
- [X] 118 個元素卡片正確顯示
- [X] Table 佈局位置正確 (18欄×10列)
- [X] 初始動畫 3 秒完成

### Tasks

- [X] T005 [US1] 定義 118 個元素的資料陣列 `table[]` 於 `examples/css3d-periodic-table/index.html`
- [X] T006 [US1] 實作元素卡片 DOM 結構 (.element, .number, .symbol, .details) 與 CSS 樣式
- [X] T007 [US1] 建立 CSS3DObject 迴圈，將 118 個卡片加入 Scene 並計算 Table 佈局目標位置
- [X] T008 [US1] 實作 `transform(targets, duration)` 函式並觸發初始動畫

**Dependencies**: Phase 2 完成

---

## Phase 4: User Story 2 - 視圖切換 [P1]

**Story**: 作為使用者，我希望能在 Table、Sphere、Helix、Grid 四種視圖間切換

**Goal**: 實作四種佈局算法與按鈕切換

**Independent Test Criteria**:
- [X] 四個按鈕可點擊
- [X] 每種視圖切換動畫流暢 (2-4秒)
- [X] 當前視圖按鈕有 disabled 狀態

### Tasks

- [X] T009 [P] [US2] 實作 Sphere 佈局算法 (球面座標公式) 於 `examples/css3d-periodic-table/index.html`
- [X] T010 [P] [US2] 實作 Helix 佈局算法 (圓柱座標公式) 於 `examples/css3d-periodic-table/index.html`
- [X] T011 [P] [US2] 實作 Grid 佈局算法 (5×5×5 立方網格) 於 `examples/css3d-periodic-table/index.html`
- [X] T012 [US2] 建立四個視圖按鈕 UI (#menu) 與事件綁定，包含 disabled 狀態管理

**Dependencies**: Phase 3 完成 (需要 transform 函式)

**Parallel Opportunities**: T009, T010, T011 可並行開發 (獨立的佈局算法)

---

## Phase 5: User Story 3 - 滑鼠互動 [P2]

**Story**: 作為使用者，我希望能用滑鼠拖曳旋轉視角、滾輪縮放，以及 hover 元素時有視覺反饋

**Goal**: 實作 TrackballControls 與 Hover 效果

**Independent Test Criteria**:
- [X] 拖曳可 360 度旋轉
- [X] 滾輪可縮放 (有範圍限制)
- [X] Hover 元素有背景色變化

### Tasks

- [X] T013 [US3] 整合 TrackballControls 於 `init()` 函式，設定 rotateSpeed、zoomSpeed、minDistance、maxDistance
- [X] T014 [P] [US3] 實作元素卡片 CSS hover 效果 (背景色、box-shadow 漸變)
- [X] T015 [US3] 確保 controls.update() 在 animate 迴圈中正確呼叫

**Dependencies**: Phase 2 完成

**Note**: 此 Phase 可與 Phase 4 並行開發

---

## Phase 6: User Story 4 - 響應式調整 [P3]

**Story**: 作為使用者，我希望在調整瀏覽器視窗大小時，3D 場景能自動適應

**Goal**: 實作 window resize 處理

**Independent Test Criteria**:
- [X] 調整視窗大小時畫面不變形
- [X] 全螢幕切換正常

### Tasks

- [X] T016 [US4] 實作 `onWindowResize()` 函式，更新 camera.aspect、projection matrix、renderer size
- [X] T017 [US4] 綁定 window resize 事件監聽器

**Dependencies**: Phase 2 完成

**Note**: 此 Phase 可與 Phase 4, 5 並行開發

---

## Phase 7: Polish (最終優化)

**Goal**: 程式碼整理、效能確認、文件更新

- [X] T018 檢查整體程式碼結構、註解，確保與 three.js 官方範例風格一致

**Completion Criteria**:
- [X] 所有功能正常運作
- [X] 60fps 效能達標
- [X] 程式碼風格一致

**Dependencies**: Phase 3-6 全部完成

---

## Dependencies Graph

```
Phase 1 (Setup)
    │
    ▼
Phase 2 (Foundational)
    │
    ├──────────────────────┬──────────────────────┐
    ▼                      ▼                      ▼
Phase 3 (US1)         Phase 5 (US3)         Phase 6 (US4)
    │                      │                      │
    ▼                      │                      │
Phase 4 (US2)              │                      │
    │                      │                      │
    └──────────────────────┴──────────────────────┘
                           │
                           ▼
                    Phase 7 (Polish)
```

---

## Parallel Execution Examples

### Parallel Group 1: 佈局算法 (Phase 4)
```
可並行: T009, T010, T011
原因: 三種佈局算法完全獨立，只需相同的輸入介面
```

### Parallel Group 2: User Story 3-6 部分任務
```
可並行: Phase 5 (US3) 與 Phase 6 (US4)
原因: 這兩個 User Story 都只依賴 Phase 2，可與 Phase 4 同時開發
```

### Parallel Group 3: 樣式與邏輯
```
可並行: T006 (CSS 樣式) 與 T005 (資料定義)
原因: 樣式定義與資料陣列無依賴關係
```

---

## Implementation Strategy

### MVP Scope (建議首次實作範圍)
- Phase 1 + Phase 2 + Phase 3 (US1)
- 交付成果: 可顯示 118 個元素的靜態週期表視圖

### Incremental Delivery
1. **Increment 1**: MVP (Table 視圖)
2. **Increment 2**: 加入 US2 (四種視圖切換)
3. **Increment 3**: 加入 US3 (滑鼠互動)
4. **Increment 4**: 加入 US4 (響應式) + Polish

### Estimated Effort
- Setup + Foundational: 15 分鐘
- US1 (Table View): 30 分鐘
- US2 (View Switching): 20 分鐘
- US3 (Hover/Controls): 15 分鐘
- US4 (Resize): 10 分鐘
- Polish: 10 分鐘
- **Total**: ~1.5-2 小時

---

## Validation Checklist

### 功能驗證
- [ ] 118 個元素全部顯示
- [ ] 四種視圖切換正常
- [ ] 動畫流暢無卡頓
- [ ] 滑鼠互動正常
- [ ] 視窗調整正常

### 效能驗證
- [ ] 維持 60fps
- [ ] 初始載入 < 2 秒
- [ ] 動畫時間符合規格

### 相容性驗證
- [ ] Chrome 最新版
- [ ] Firefox 最新版
- [ ] Safari 最新版
- [ ] Edge 最新版
