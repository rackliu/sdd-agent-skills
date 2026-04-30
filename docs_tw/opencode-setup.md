# OpenCode 設定指南

本指南說明如何在 OpenCode 中使用 Agent Skills，並盡可能貼近 Claude Code 的體驗（自動選擇 skill、依生命週期驅動工作流程，以及嚴格的流程約束）。

## 概觀

OpenCode 支援自訂 `/commands`，但不像 Claude Code 那樣具備原生外掛系統或自動 skill 路由機制。

因此，我們是透過以下方式達成近似效果：

- 強化的 system prompt（`AGENTS.md`）
- 內建的 `skill` 工具
- 從 `/skills` 目錄進行一致性的 skill 探索

這樣可以建立一種**由 agent 驅動的工作流程**，讓 skills 被自動選擇與執行。

雖然你也可以在 OpenCode 中自行重建 `/spec`、`/plan` 等命令，但這套整合刻意改採 agent-driven 的方式：

- 根據使用者意圖自動選擇 skills
- 透過 `AGENTS.md` 強制落實工作流程
- 不需要手動呼叫命令

這比手動命令更貼近 Claude Code 的實際行為，因為 Claude Code 通常是自動觸發 skills，而不是由使用者手動啟動。

---

## 安裝

1. 複製此儲存庫：

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

2. 在 OpenCode 中開啟專案。

3. 確認你的 workspace 中包含以下檔案：

- `AGENTS.md`（根目錄）
- `skills/` 目錄

不需要額外安裝其他內容。

---

## 運作方式

### 1. Skill 探索

所有 skills 都放在：

```
skills/<skill-name>/SKILL.md
```

OpenCode agents 會依照 `AGENTS.md` 指示：

- 偵測何時該使用某個 skill
- 呼叫 `skill` 工具
- 嚴格依照 skill 執行

### 2. 自動呼叫 Skills

agent 會評估每一個請求，並映射到合適的 skill。

範例：

- 「build a feature」→ `incremental-implementation` + `test-driven-development`
- 「design a system」→ `spec-driven-development`
- 「fix a bug」→ `debugging-and-error-recovery`
- 「review this code」→ `code-review-and-quality`

使用者**不需要**明確指定要用哪個 skill。

### 3. 生命週期映射（隱式 Commands）

開發生命週期會以隱式方式編碼：

- DEFINE → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development`
- VERIFY → `debugging-and-error-recovery`
- REVIEW → `code-review-and-quality`
- SHIP → `shipping-and-launch`

這就取代了 `/spec`、`/plan` 等 slash commands。

---

## 使用範例

### 範例 1：功能開發

使用者：
```
Add authentication to this app
```

Agent 行為：
- 偵測這是功能開發工作
- 呼叫 `spec-driven-development`
- 在寫程式碼前先產出 spec
- 再進入規劃與實作相關 skills

---

### 範例 2：修 bug

使用者：
```
This endpoint is returning 500 errors
```

Agent 行為：
- 呼叫 `debugging-and-error-recovery`
- 重現 → 定位 → 修正 → 補上防護

---

### 範例 3：程式碼審查

使用者：
```
Review this PR
```

Agent 行為：
- 呼叫 `code-review-and-quality`
- 套用結構化審查（正確性、設計、可讀性等）

---

## Agent 預期行為（關鍵）

若要讓 OpenCode 正常運作，agent 必須遵守以下規則：

- 在採取任何動作前，先檢查是否有適用的 skill
- 只要 skill 適用，就**必須**使用
- 不可跳過必要流程（spec、plan、test 等）
- 不可直接跳進實作

這些規則都會由 `AGENTS.md` 強制約束。

---

## 限制

- 沒有原生 slash commands（改以意圖映射處理）
- 沒有外掛系統（改以 prompt + 結構處理）
- Skill 呼叫仍依賴模型是否確實遵循規則

即使有這些限制，整體工作流程在實務上仍與 Claude Code 非常接近。

---

## 建議工作方式

直接使用自然語言即可：

- 「設計一個功能」
- 「規劃這項變更」
- 「實作這個需求」
- 「修正這個 bug」
- 「審查這份程式碼」

agent 會自動選擇並執行正確的 skills。

---

## 總結

OpenCode 整合的核心做法是結合：

- 結構化 skills（本儲存庫）
- 強力的 agent 規則（`AGENTS.md`）
- 透過推理進行自動 skill 呼叫

最終可形成一套**完全由 agent 驅動、可用於正式工程環境的工作流程**，而且不需要外掛或手動命令。
