# Spec Kit 範本文件完整指南

> **📚 內容出處**
> 本文件轉自：https://sdd.gh.miniasp.com/labs/claude-projects

---

## 概述

本指南詳細說明 Spec Kit 中各個範本檔案的結構和使用方法，包括憲法產生器、計畫範本、規格範本和任務範本。這些範本是規格驅動開發 (SDD) 的核心工具。

---

## 1. Spec Kit 憲法產生器

### 功能名稱

**Spec Kit Constitution Generator**

### 描述

從互動式或提供的原則輸入建立或更新專案憲法，確保所有相依的範本保持同步。

### 使用方法

#### 基本用法

```powershell
/speckit.constitution
```

#### 命令範本

斜杠命令：`/speckit.constitution`

```
<speckit.constitution>
---
description: Create or update the project constitution from interactive or provided principle inputs, ensuring all dependent templates stay in sync.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).
```

### 憲法生成流程

#### 1. 載入現有憲法文件

從 `.specify/memory/constitution.md` 載入現有憲法檔案。
- 識別每個佔位符令牌（例如 `[PROJECT_NAME]`, `[PRINCIPLE_1_NAME]`）
- **重要**：使用者可能需要不同數量的原則。如果指定了數字，請尊重該數字

#### 2. 收集/衍生佔位符值

- 如果使用者輸入（會話）提供值，使用它
- 否則從現有 repo 上下文推斷（README、文件、先前的憲法版本）
- 治理日期：`RATIFICATION_DATE` 是原始採用日期（如果未知則詢問或標記 TODO），`LAST_AMENDED_DATE` 是今天（如果進行了更改），否則保留之前的
- `CONSTITUTION_VERSION` 必須根據語義版本控制規則遞增：
  - **MAJOR**：不兼容的治理/原則移除或重新定義
  - **MINOR**：新增或大幅擴展指導的原則/章節
  - **PATCH**：澄清、措辭、打字修正、非語義完善

#### 3. 起草更新的憲法內容

- 使用具體文字替換每個佔位符（除了意圖保留的範本槽之外沒有方括號令牌）
- 保留標題層級和註釋可在替換後移除（除非仍然提供澄清指導）
- 確保每個原則部分包括：簡潔的名稱行、捕捉非協商規則的段落（或項目符號列表）、明確的理由（如果不明顯）
- 確保治理部分列出修訂程序、版本控制政策和合規審查期望

#### 4. 一致性傳播檢查清單

- 讀取 `.specify/templates/plan-template.md`，確保任何「憲法檢查」或規則與更新的原則一致
- 讀取 `.specify/templates/spec-template.md` 的範圍/需求對齊——如果憲法添加/移除強制部分或約束，則更新
- 讀取 `.specify/templates/tasks-template.md`，確保任務分類反映新增或移除的原則驅動任務類型
- 讀取任何執行時間指導文件（例如 `README.md`、`docs/quickstart.md` 或特定於代理的指導檔案）。更新改變的原則的引用

#### 5. 生成同步影響報告

在憲法檔案更新後的頂部添加 HTML 註釋：

- 版本變更：舊 → 新
- 修改的原則清單（舊標題 → 新標題（如果重新命名））
- 新增章節
- 移除章節
- 需要更新的範本
- 如果有任何佔位符有意延期，則跟進 TODO

#### 6. 最終驗證

- 沒有剩余未解釋的方括號令牌
- 版本行匹配報告
- 日期為 ISO 格式 YYYY-MM-DD
- 原則是陳述性的、可測試的、沒有模糊語言

#### 7. 寫回並輸出總結

- 將完成的憲法寫回 `.specify/memory/constitution.md`
- 輸出最終摘要包括：
  - 新版本和凹版理由
  - 任何標記手動跟進的檔案
  - 建議的提交訊息

---

## 2. 計畫範本 (plan-template.md)

### 檔案位置

`.specify/templates/plan-template.md`

### 範本結構

#### 基本資訊

```
# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`
```

#### 核心部分

1. **摘要** - 特性需求 + 技術方法
2. **技術上下文**
   - 語言/版本
   - 主要相依性
   - 儲存體
   - 測試
   - 目標平台
   - 專案類型
   - 效能目標
   - 約束條件
   - 規模/範圍

3. **憲法檢查** - 必須在 Phase 0 研究前通過，在 Phase 1 設計後重新檢查

4. **專案結構**
   - 文件結構
   - 源代碼結構選項
   - 結構決定說明

5. **複雜性追蹤** - 只有在憲法檢查有違規時才填充

---

## 3. 規格範本 (spec-template.md)

### 檔案位置

`.specify/templates/spec-template.md`

### 範本結構

#### 基本資訊

```markdown
# Feature Specification: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`
**Created**: [DATE]
**Status**: Draft
**Input**: User description: "$ARGUMENTS"
```

#### 主要部分

### 1. 用戶場景與測試 (必需)

#### 用戶故事格式

```markdown
### User Story 1 - [Brief Title] (Priority: P1)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this can be tested independently]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]
2. **Given** [initial state], **When** [action], **Then** [expected outcome]
```

#### 優先級說明

- **P1** - 最關鍵的故事，構成 MVP
- **P2** - 重要但不影響基本功能
- **P3** - 可選增強

#### 邊界情況

```markdown
### Edge Cases

- What happens when [boundary condition]?
- How does system handle [error scenario]?
```

### 2. 需求 (必需)

#### 功能需求

```markdown
### Functional Requirements

- **FR-001**: System MUST [specific capability]
- **FR-002**: System MUST [specific capability]
- **FR-003**: Users MUST be able to [key interaction]
- **FR-004**: System MUST [data requirement]
- **FR-005**: System MUST [behavior]
```

不清楚的需求標記：

```markdown
- **FR-006**: System MUST authenticate users via [NEEDS CLARIFICATION: auth method not specified]
```

#### 關鍵實體

```markdown
### Key Entities

- **[Entity 1]**: [What it represents, key attributes without implementation]
- **[Entity 2]**: [What it represents, relationships to other entities]
```

### 3. 成功標準 (必需)

```markdown
## Success Criteria

### Measurable Outcomes

- **SC-001**: [Measurable metric, e.g., "Users can complete task in under 2 minutes"]
- **SC-002**: [Measurable metric, e.g., "System handles 1000 concurrent users"]
- **SC-003**: [User satisfaction metric, e.g., "90% success rate on first attempt"]
- **SC-004**: [Business metric, e.g., "Reduce support tickets by 50%"]
```

---

## 4. 任務範本 (tasks-template.md)

### 檔案位置

`.specify/templates/tasks-template.md`

### 任務格式

```
[ID] [P?] [Story] Description
```

- **[ID]** - 任務編號
- **[P?]** - 可以並行運行的任務（不同檔案，無相依性）
- **[Story]** - 該任務屬於的用戶故事
- 在描述中包含確切的檔案路徑

### 路徑約定

- **單一專案**：`src/`、`tests/` 在存儲庫根目錄
- **Web 應用**：`backend/src/`、`frontend/src/`
- **行動應用**：`api/src/`、`ios/src/` 或 `android/src/`

### 階段結構

#### Phase 1: Setup (共享基礎設施)

```markdown
## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project structure per implementation plan
- [ ] T002 Initialize [language] project with [framework] dependencies
- [ ] T003 [P] Configure linting and formatting tools
```

#### Phase 2: Foundational (阻塞前提條件)

```markdown
## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story

⚠️ CRITICAL: No user story work can begin until this phase is complete

- [ ] T004 Setup database schema and migrations framework
- [ ] T005 [P] Implement authentication/authorization framework
- [ ] T006 [P] Setup API routing and middleware structure
- [ ] T007 Create base models/entities that all stories depend on
- [ ] T008 Configure error handling and logging infrastructure
- [ ] T009 Setup environment configuration management

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel
```

#### Phase 3+: 用戶故事

每個用戶故事包括：

```markdown
## Phase 3: User Story 1 - [Title] (Priority: P1) 🎯 MVP

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 1 (OPTIONAL)

- [ ] T010 [P] [US1] Contract test for [endpoint]
- [ ] T011 [P] [US1] Integration test for [user journey]

### Implementation for User Story 1

- [ ] T012 [P] [US1] Create [Entity1] model in src/models/[entity1].py
- [ ] T013 [P] [US1] Create [Entity2] model in src/models/[entity2].py
- [ ] T014 [US1] Implement [Service] in src/services/[service].py
- [ ] T015 [US1] Implement [endpoint/feature]

**Checkpoint**: User Story 1 should be fully functional and independently testable
```

### 相依性與執行順序

#### Phase 相依性

- **Setup (Phase 1)** - 無相依性，立即開始
- **Foundational (Phase 2)** - 依賴 Setup 完成 - **阻塞所有用戶故事**
- **用戶故事 (Phase 3+)** - 所有取決於 Foundational 完成
- **Polish (最終)** - 取決於所有所需的用戶故事完成

#### 用戶故事相依性

- **US1 (P1)** - 可在 Foundational 後開始 - 無其他故事相依性
- **US2 (P2)** - 可在 Foundational 後開始 - 可與 US1 集成但應獨立可測試
- **US3 (P3)** - 可在 Foundational 後開始 - 可與 US1/US2 集成但應獨立可測試

#### 並行機會

```bash
# 所有 Setup 標記 [P] 的任務可並行運行
# 所有 Foundational 標記 [P] 的任務可並行運行
# Foundational 完成後，所有用戶故事可並行開始
# 同一用戶故事中標記 [P] 的模型可並行
```

### MVP 優先策略

1. 完成 Phase 1: Setup
2. 完成 Phase 2: Foundational
3. 完成 Phase 3: User Story 1
4. **停止並驗證** - 獨立測試 US1
5. 根據需要部署/演示

### 增量交付

1. Setup + Foundational → 基礎就緒
2. 新增 US1 → 測試獨立 → 部署/演示 (MVP!)
3. 新增 US2 → 測試獨立 → 部署/演示
4. 新增 US3 → 測試獨立 → 部署/演示

---

## 5. 最佳實踐

### 範本使用檢查清單

- [ ] 憲法文件清晰且最新
- [ ] 計畫範本包括完整的技術上下文
- [ ] 規格範本列出所有用戶故事和成功標準
- [ ] 任務範本按用戶故事組織並獨立可測試
- [ ] 所有依賴性明確文件化
- [ ] 可以並行的任務正確標記 [P]

### 常見陷阱

❌ **避免**：
- 不清楚的需求標記
- 缺失的成功標準
- 跨故事的緊密耦合
- 不明確的任務優先級

✅ **做**：
- 保持故事獨立且可測試
- 清晰標記相依性
- 定期更新版本控制
- 在每個檢查點驗證

---

## 6. 資源與參考

### 相關連結

- [規格驅動開發實戰：AI 時代的軟體開發新典範](https://sdd.gh.miniasp.com/)

### 聯繫方式

多奇教育訓練 • 2025

Powered by [Beautiful Jekyll](https://beautifuljekyll.com/)
