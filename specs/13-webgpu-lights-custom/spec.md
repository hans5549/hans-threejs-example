# Feature Specification: WebGPU Custom Lighting Model

**Feature Branch**: `13-webgpu-lights-custom`  
**Created**: 2025-12-01  
**Status**: Draft  
**Input**: User description: "Implement WebGPU custom lighting model example with Three.js, featuring selective lights and custom lighting model for point cloud visualization"

## Overview

本功能實作一個 Three.js WebGPU 自訂光照模型範例，展示如何使用 WebGPU 渲染器和 Three.js Shading Language (TSL) 建立自訂光照效果。此範例包含選擇性光源控制、自訂光照模型類別，以及大量粒子（50 萬點）的即時光照視覺化。

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 觀看動態光源點雲效果 (Priority: P1)

使用者開啟網頁後，能立即看到一個由 50 萬個粒子組成的點雲，被三個不同顏色的動態光源（橙色、藍色、綠色）照亮。光源會以不同的軌跡在空間中移動，產生流動的光影效果。

**Why this priority**: 這是核心視覺體驗，展示 WebGPU 自訂光照模型的主要功能價值。

**Independent Test**: 開啟網頁即可看到動態光照效果，無需任何使用者操作。

**Acceptance Scenarios**:

1. **Given** 使用者在支援 WebGPU 的瀏覽器中, **When** 開啟範例頁面, **Then** 看到由三個彩色動態光源照亮的點雲
2. **Given** 頁面已載入, **When** 觀察畫面數秒, **Then** 可見光源在空間中以不同軌跡平滑移動
3. **Given** 點雲正在顯示, **When** 光源靠近某區域的粒子, **Then** 該區域粒子的顏色會根據光源顏色變亮

---

### User Story 2 - 互動式場景導覽 (Priority: P2)

使用者能透過滑鼠拖曳旋轉視角、滾輪縮放距離，從不同角度觀察點雲和光源的互動效果。

**Why this priority**: 提供使用者主動探索的能力，增強理解和沉浸感。

**Independent Test**: 使用滑鼠操作即可驗證視角控制功能。

**Acceptance Scenarios**:

1. **Given** 畫面已顯示, **When** 使用者拖曳滑鼠左鍵, **Then** 視角圍繞場景中心旋轉
2. **Given** 畫面已顯示, **When** 使用者滾動滑鼠滾輪, **Then** 相機縮放，距離在 0 到 4 單位之間
3. **Given** 使用者正在旋轉視角, **When** 放開滑鼠, **Then** 視角停止旋轉並維持當前位置

---

### User Story 3 - 響應式視窗調整 (Priority: P3)

當使用者調整瀏覽器視窗大小時，畫面會自動調整以填滿整個視窗，保持正確的長寬比例。

**Why this priority**: 確保各種螢幕尺寸下的正確顯示，是基本的使用者體驗要求。

**Independent Test**: 調整瀏覽器視窗大小即可驗證。

**Acceptance Scenarios**:

1. **Given** 範例正在執行中, **When** 使用者調整瀏覽器視窗大小, **Then** 渲染畫面自動填滿視窗並保持正確比例
2. **Given** 視窗已調整大小, **When** 繼續觀看動畫, **Then** 動畫流暢播放無中斷

---

### Edge Cases

- 當瀏覽器不支援 WebGPU 時，應顯示適當的提示訊息或降級處理
- 當視窗縮到極小尺寸時，畫面應仍能正常渲染
- 當使用者快速連續調整視窗大小時，不應造成效能問題或畫面閃爍

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 使用 WebGPU 渲染器進行畫面渲染
- **FR-002**: 系統 MUST 實作自訂光照模型類別，繼承自 Three.js LightingModel
- **FR-003**: 系統 MUST 建立三個不同顏色的點光源（橙色 0xffaa00、藍色 0x0040ff、綠色 0x80ff80）
- **FR-004**: 系統 MUST 使用選擇性光源節點（selective lights node）將特定光源應用於點雲材質
- **FR-005**: 系統 MUST 產生 50 萬個隨機分佈的 3D 點組成點雲
- **FR-006**: 系統 MUST 為每個光源建立一個可見的小球體 mesh 表示其位置
- **FR-007**: 系統 MUST 實作光源的動態移動動畫，使用三角函數產生平滑軌跡
- **FR-008**: 系統 MUST 整合 OrbitControls 提供使用者視角控制
- **FR-009**: 系統 MUST 監聽視窗 resize 事件並更新相機和渲染器尺寸
- **FR-010**: 系統 MUST 以持續的動畫迴圈（animation loop）更新場景
- **FR-011**: 光源小球 MUST 使用空的 lightsNode 以忽略場景光源影響，保持自發光效果

### Key Entities

- **CustomLightingModel**: 自訂光照模型類別，定義光照計算邏輯
- **PointLight**: 三個動態點光源，各有不同顏色和移動軌跡
- **Points (Point Cloud)**: 由 50 萬個頂點組成的粒子系統
- **PointsNodeMaterial**: 使用自訂光照模型的點雲材質
- **Light Sphere Mesh**: 表示光源位置的小球體視覺指示器

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 頁面載入後 3 秒內呈現完整的動態光照效果
- **SC-002**: 點雲包含 50 萬個粒子且動畫流暢（目標 60 FPS，最低 30 FPS）
- **SC-003**: 三個光源同時可見且各自以不同軌跡移動
- **SC-004**: 使用者可在 360 度範圍內自由旋轉視角觀察場景
- **SC-005**: 視窗調整大小後 0.5 秒內畫面自動適應新尺寸
- **SC-006**: 光源接近粒子時，粒子顏色變化清晰可見

## Assumptions

- 使用者的瀏覽器支援 WebGPU（Chrome 113+、Edge 113+ 或其他支援 WebGPU 的瀏覽器）
- 使用者的裝置具備基本的 GPU 能力以處理 50 萬粒子的渲染
- Three.js 版本支援 WebGPU 渲染器和 TSL（Three.js Shading Language）
- 本專案使用 CDN 或本地 ES modules 載入 Three.js

## Out of Scope

- WebGL 向下相容（本範例專注於 WebGPU）
- 光源參數的 GUI 控制介面
- 粒子數量的動態調整
- 儲存或分享場景狀態
- 觸控裝置的特殊手勢支援（依賴 OrbitControls 的預設觸控支援）
