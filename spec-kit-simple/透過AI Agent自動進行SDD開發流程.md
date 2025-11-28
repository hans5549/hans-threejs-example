# 透過 AI Agent 自動進行整個 SDD 開發流程

> **📚 內容出處**
> 本文件轉自：https://sdd.gh.miniasp.com/labs/ai-agent

---

## 概述

本指南介紹如何使用 AI Agent 自動執行規格驅動開發 (Specification-Driven Development, SDD) 的完整流程。透過配置合適的 AI 模型和任務設定，可以大幅提升開發效率。

---

## 1. 準備 AI Agent

### 使用 Copilot CLI

```powershell
copilot --allow-all-tools --allow-all-paths --model claude-haiku-4.5
```

### 使用 Gemini CLI

```bash
gemini -y
```

---

## 2. 設定 Agent 任務

### 規劃階段設定

規劃階段建議採用 `claude-sonnet-4.5` 模型。

Agent 任務配置如下：

```
你是一個負責執行 Spec Kit 流程的 AI 助手，會主動幫我叫用 `copilot` 命令完成任務，你主要有以下任務：

1. 先整理用戶的提示詞，寫入 `PROMPT.txt` 檔案

2. 幫用戶執行 `copilot` 命令，執行的命令範本如下：

   若是第一次執行，命令如下：

   type PROMPT.txt | copilot --allow-all-tools --allow-all-paths --model claude-sonnet-4.5

   若是之後的命令，命令如下：

   type PROMPT.txt | copilot --allow-all-tools --allow-all-paths --model claude-sonnet-4.5 --continue

3. 當執行 `copilot` 命令結束後，撰寫一份 zh-tw 的摘要總結當成 commit 訊息，並且直接將變更 commit 進版控

4. 如果任務尚未完成，並且 copilot 有建議的下一步，只要不是進入下一個階段，那就請繼續執行！

5. 請不要直接執行下一個 Slash command，若嘗試要執行下一個 Slash command，請暫時停止你的任務。

請你先告訴我你的理解。
```

### 實作階段設定

實作階段建議採用 `claude-haiku-4.5` 模型。

```
你是一個負責執行 Spec Kit 流程的 AI 助手，會主動幫我叫用 `copilot` 命令完成任務，你主要有以下任務：

1. 先整理用戶的提示詞，寫入 `PROMPT.txt` 檔案

2. 幫用戶執行 `copilot` 命令，執行的命令範本如下：

   若是第一次執行，命令如下：

   type PROMPT.txt | copilot --allow-all-tools --allow-all-paths --model claude-haiku-4.5

   若是之後的命令，命令如下：

   type PROMPT.txt | copilot --allow-all-tools --allow-all-paths --model claude-haiku-4.5 --continue

3. 當執行 `copilot` 命令結束後，撰寫一份 zh-tw 的摘要總結當成 commit 訊息，並且直接將變更 commit 進版控

4. 如果任務尚未完成，並且 copilot 有建議的下一步，只要不是進入下一個階段，那就請繼續執行！

5. 請不要直接執行下一個 Slash command，若嘗試要執行下一個 Slash command，請暫時停止你的任務。
```

---

## 3. SDD 開發流程步驟

### Step 1: 指定規格

執行規格制定命令：

```powershell
<PROMPT>
/speckit.specify
# 會員註冊流程

我們的核心目標是讓新使用者能透過一個簡單、安全的流程，順利註冊帳號並完成 E-Mail 驗證，以便使用平台功能。

整個註冊是一個單向流程，無法返回上一步。使用者首先需要填寫身分證字號、姓名等資料，並設定一組 8 到 20 碼、包含英文大小寫與數字的密碼。系統會檢查身分證字號，防止重複註冊。

送出資料後，系統會寄出一封內含 6 位數驗證碼的 E-Mail，有效時間為 5 分鐘。使用者必須手動輸入驗證碼來完成驗證。

如果使用者未完成 E-Mail 驗證，依然可以用剛設定的帳號密碼登入，但部分功能將受限。直到輸入正確的驗證碼後，帳號所有功能才會被啟用。註冊完成後，系統不會自動登入。

為了讓流程單純，我們不做社群帳號註冊、不提供密碼強度提示，也不支援點擊 E-Mail 連結直接驗證。
</PROMPT>
```

### Step 2: 釐清問題

```powershell
<PROMPT>
/speckit.clarify
</PROMPT>
```

### Step 3: 制定開發計畫

#### ASP.NET Core 8.0 範例

```powershell
<PROMPT>
/speckit.plan
We are going to generate this using ASP.NET Core 8.0 Web API, using SQL Server as the database. This project is mainly for backend REST API only. No frontend implementation is required. Use EF Core Code First workflow.
I don't want to use AutoMapper to map DTO. Use POCO instead.
I don't want to use Redis.
I don't want to use Minimal APIs.
</PROMPT>
```

#### Node.js 範例

```powershell
<PROMPT>
/speckit.plan
We are going to generate this using Node.js and Express, using SQL Server as the database. This project is mainly for backend REST API only. No frontend implementation is required.
</PROMPT>
```

### Step 4: 規劃任務清單

```powershell
<PROMPT>
/speckit.tasks
</PROMPT>
```

### Step 5: 產生規格分析報告

```powershell
<PROMPT>
/speckit.analyze Save your analyze report to `analyze-01.md`
</PROMPT>
```

### Step 6: 開始實作

如果實作必須分段進行，請一直執行到最後完成：

```powershell
<PROMPT>
/speckit.implement
</PROMPT>
```

---

## 4. AI 模型選擇建議

| 階段 | 建議模型 | 說明 |
|------|---------|------|
| 規劃階段 | claude-sonnet-4.5 | 更強大的推理能力，適合複雜的規格制定和計畫 |
| 實作階段 | claude-haiku-4.5 | 輕量高效，適合代碼生成和具體實作任務 |

---

## 5. 最佳實踐

### Agent 自動化的優勢

1. **連續執行** - Agent 自動執行一系列命令，無需手動干預
2. **版本控制集成** - 每個階段完成後自動生成 commit 訊息並提交
3. **智能決策** - Agent 根據執行結果判斷是否繼續或停止
4. **階段管理** - 清晰的階段划分，避免跨越流程邊界

### 注意事項

- 不要跳過任何階段直接進入下一個 Slash command
- 如果任務未完成但有建議的下一步，應繼續執行
- 規劃和實作階段使用不同的模型配置
- 每個階段完成後，確保 commit 訊息清晰準確

---

## 6. 其他資源

### 相關連結

- [規格驅動開發實戰：AI 時代的軟體開發新典範](https://sdd.gh.miniasp.com/)
- [Copilot CLI 文件](https://github.com/github/copilot)

### 聯繫方式

多奇教育訓練 • 2025

Powered by [Beautiful Jekyll](https://beautifuljekyll.com/)
