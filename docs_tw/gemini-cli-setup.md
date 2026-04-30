# 在 Gemini CLI 中使用 agent-skills

## 設定

### 方案 1：安裝為 Skills（建議）

Gemini CLI 具有原生 skills 系統，會自動探索 `.gemini/skills/` 或 `.agents/skills/` 目錄中的 `SKILL.md` 檔案。每個 skill 都會在與你的任務相符時按需啟用。

**從儲存庫安裝：**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**或從本機複製版本安裝：**

```bash
git clone https://github.com/addyosmani/agent-skills.git
gemini skills install /path/to/agent-skills/skills/
```

**僅安裝到特定工作區：**

```bash
gemini skills install /path/to/agent-skills/skills/ --scope workspace
```

安裝在 workspace scope 的 skills 會放進 `.gemini/skills/`（或 `.agents/skills/`）。使用者層級的 skills 則會放進 `~/.gemini/skills/`。

安裝完成後，可用以下指令驗證：

```
/skills list
```

Gemini CLI 會自動把 skill 名稱與描述注入提示中。當它辨識到相符任務時，會先詢問你是否同意啟用該 skill，之後才載入完整指示。

### 方案 2：GEMINI.md（持久上下文）

如果你希望某些 skills 永遠作為專案持久上下文載入，而不是按需啟用，可將它們加入專案的 `GEMINI.md`：

```bash
# 建立 GEMINI.md，將核心 skills 作為持久上下文
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md > GEMINI.md
echo -e "\n---\n" >> GEMINI.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> GEMINI.md
```

你也可以透過匯入分離檔案的方式模組化：

```markdown
# 專案指示

@skills/test-driven-development/SKILL.md
@skills/incremental-implementation/SKILL.md
```

使用 `/memory show` 驗證已載入的上下文，並在變更後用 `/memory reload` 重新整理。

> **Skills 與 GEMINI.md 的差異：** Skills 是按需啟用的專業能力，只會在相關時載入，可讓上下文視窗保持乾淨。GEMINI.md 則是每次提示都會載入的持久上下文。階段性工作流程建議用 skills，永遠適用的專案慣例則放在 GEMINI.md。

## 建議設定

### 永遠開啟（GEMINI.md）

把下列內容作為每次 session 都會載入的持久上下文：

- `incremental-implementation` — 以小而可驗證的切片方式建構
- `code-review-and-quality` — 五面向審查

### 按需啟用（Skills）

將下列項目安裝成 skills，讓它們只在相關時啟用：

- `test-driven-development` — 在實作邏輯或修 bug 時啟用
- `spec-driven-development` — 在開始新專案或新功能時啟用
- `frontend-ui-engineering` — 在建構 UI 時啟用
- `security-and-hardening` — 在安全審查時啟用
- `performance-optimization` — 在處理效能工作時啟用

## 進階設定

### MCP 整合

這個 skill 套件中的許多 skill 都會利用 [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) 工具與環境互動。例如：

- `browser-testing-with-devtools` 會使用 `chrome-devtools` MCP 擴充。
- `performance-optimization` 可從效能相關 MCP 工具中獲益。

若要啟用它們，請確認你已在 Gemini CLI 設定檔 `~/.gemini/config.json` 中安裝對應的 MCP 擴充。

### Session Hooks

Gemini CLI 支援 session 生命週期 hooks。你可以用它們在 session 開始時自動注入上下文，或執行驗證腳本。

若要重現其他工具中的 agent-skills 體驗，可設定 `SessionStart` hook，在 session 開始時提醒可用 skills，或載入一個 meta-skill。

### 明確載入上下文

你也可以在提示中使用 `@` 符號，明確把任意 skill 載入目前 session：

```markdown
Use the @skills/test-driven-development/SKILL.md skill to implement this fix.
```

當你想確保一定遵循特定工作流程，而不想等待自動探索時，這種方式很有用。

## Slash Commands

此儲存庫在 `.gemini/commands/` 下提供 7 個 slash commands，對應完整的開發生命週期。當你從專案根目錄執行 Gemini CLI 時，它會自動探索這些命令。

| Command | 功能說明 |
|---------|----------|
| `/spec` | 在寫程式碼前先撰寫結構化 spec |
| `/planning` | 將工作拆解成小而可驗證的任務 |
| `/build` | 以漸進方式實作下一個任務 |
| `/test` | 執行 TDD 流程：red、green、refactor |
| `/review` | 五面向程式碼審查 |
| `/code-simplify` | 在不改變行為下降低複雜度 |
| `/ship` | 透過平行 persona fan-out 執行上線前檢查 |

每個命令都會自動呼叫對應 skill，不需要手動載入。

> **注意：** 請使用 `/planning` 而不是 `/plan`，因為 `/plan` 會和 Gemini CLI 內建命令名稱衝突。

## 使用秘訣

1. **優先使用 skills，而不是 GEMINI.md** — Skills 會按需啟用，能讓上下文視窗更聚焦。只有在你真的希望某個 skill 永遠載入時，才放進 GEMINI.md。
2. **Skill 描述非常重要** — 每個 `SKILL.md` 的 frontmatter 都有 `description` 欄位，用來告訴 agents 何時應該啟用它。本儲存庫中的描述已針對所有支援工具（Claude Code、Gemini CLI 等）的自動探索做過最佳化，會明確說明 skill 做什麼，以及什麼時候應該觸發。
3. **審查時使用 agents** — 需要結構化程式碼審查時，可複製 `agents/code-reviewer.md` 的內容來使用。
4. **搭配 references** — 在測試、效能等特定品質面向工作時，可同時引用 `references/` 內的 checklist。
