# agent-skills 快速開始指南

agent-skills 可搭配任何接受 Markdown 指示的 AI 程式設計 agent 使用。本指南說明通用作法；若你需要工具專屬設定，請參考對應的專用指南。

## Skills 如何運作

每個 skill 都是一份 Markdown 檔案（`SKILL.md`），描述一個特定的工程工作流程。當它被載入 agent 的上下文後，agent 就會依照該流程執行，包括驗證步驟、應避免的反模式，以及完成條件。

**Skills 不是參考文件。** 它們是 agent 要遵循的 step-by-step 流程。

## 快速開始（適用任何 Agent）

### 1. 複製此儲存庫

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

### 2. 選擇一個 skill

瀏覽 `skills/` 目錄。每個子目錄都包含一份 `SKILL.md`，內容包括：
- **何時使用** — 表示此 skill 適用的觸發條件
- **流程** — step-by-step 工作流程
- **驗證** — 如何確認工作已完成
- **常見合理化藉口** — agent 可能用來跳過步驟的理由
- **紅旗訊號** — 表示 skill 正被違反的跡象

### 3. 將 skill 載入你的 agent

把對應的 `SKILL.md` 內容複製到你的 agent system prompt、rules 檔案或對話中。最常見的方式如下：

**System prompt：** 在 session 一開始貼上 skill 內容。

**Rules 檔案：** 把 skill 內容加入專案的 rules 檔案（例如 CLAUDE.md、.cursorrules 等）。

**對話：** 在下指示時引用 skill，例如：「請依照 test-driven-development 流程處理這次變更。」

### 4. 用 meta-skill 協助探索

一開始可先載入 `using-agent-skills` skill。它內含一個流程圖，可把任務類型映射到適合的 skill。

## 建議設定

### 最小組合（先從這裡開始）

將以下三個核心 skills 載入你的 rules 檔案：

1. **spec-driven-development** — 用來定義要建構什麼
2. **test-driven-development** — 用來證明功能正確
3. **code-review-and-quality** — 用來在合併前驗證品質

這三個 skill 能覆蓋 AI 輔助開發中最關鍵的品質缺口。

### 完整生命週期

若要更完整的覆蓋範圍，可依開發階段載入 skills：

```
開始一個專案：  spec-driven-development → planning-and-task-breakdown
開發期間：      incremental-implementation + test-driven-development
合併前：        code-review-and-quality + security-and-hardening
部署前：        shipping-and-launch
```

### 依情境載入

不要一次載入所有 skills，這會浪費上下文。請依目前任務載入相關 skills：

- 正在做 UI？載入 `frontend-ui-engineering`
- 正在除錯？載入 `debugging-and-error-recovery`
- 正在設定 CI？載入 `ci-cd-and-automation`

## Skill 結構

每個 skill 都遵循相同結構：

```
YAML frontmatter（name、description）
├── Overview — 這個 skill 在做什麼
├── When to Use — 觸發條件與適用情境
├── Core Process — step-by-step 工作流程
├── Examples — 程式碼範例與模式
├── Common Rationalizations — 常見藉口與反駁
├── Red Flags — skill 被違反的跡象
└── Verification — 完成條件檢查清單
```

完整規格請參考 [skill-anatomy.md](skill-anatomy.md)。

## 使用 Agents

`agents/` 目錄中包含已預先設定好的 agent personas：

| Agent | 用途 |
|-------|------|
| `code-reviewer.md` | 五面向程式碼審查 |
| `test-engineer.md` | 測試策略與測試撰寫 |
| `security-auditor.md` | 漏洞偵測 |

當你需要專門化審查時，就載入對應 agent 定義。舉例來說，你可以要求你的程式設計 agent「使用 code-reviewer agent persona 審查這次變更」，並提供該 agent 定義。

## 使用 Commands

`.claude/commands/` 目錄提供 Claude Code 可用的 slash commands：

| Command | 會呼叫的 Skill |
|---------|----------------|
| `/spec` | spec-driven-development |
| `/plan` | planning-and-task-breakdown |
| `/build` | incremental-implementation + test-driven-development |
| `/test` | test-driven-development |
| `/review` | code-review-and-quality |
| `/ship` | shipping-and-launch |

## 使用 References

`references/` 目錄中提供補充用 checklist：

| Reference | 建議搭配 |
|-----------|----------|
| `testing-patterns.md` | test-driven-development |
| `performance-checklist.md` | performance-optimization |
| `security-checklist.md` | security-and-hardening |
| `accessibility-checklist.md` | frontend-ui-engineering |

當你需要 skill 以外的更細節模式時，可載入對應 reference。

## Spec 與任務產物

`/spec` 與 `/plan` 命令會建立工作中的產物（`SPEC.md`、`tasks/plan.md`、`tasks/todo.md`）。在工作進行期間，請把它們當成**活文件**：

- 在開發過程中保留於版本控制中，讓人類與 agent 共享同一份真實來源。
- 當範圍或決策改變時同步更新。
- 如果你的儲存庫不希望長期保留這些檔案，可在 merge 前刪除，或把資料夾加入 `.gitignore`；工作流程本身不要求它們永久存在。

## 使用秘訣

1. **任何非 trivial 工作，都先從 spec-driven-development 開始**
2. **寫程式碼時永遠載入 test-driven-development**
3. **不要跳過驗證步驟** — 這正是整套方法的核心價值
4. **選擇性載入 skills** — 上下文不是越多越好
5. **審查時使用 agents** — 不同視角會抓到不同問題
