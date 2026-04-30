# 在 GitHub Copilot 中使用 agent-skills

## 設定

### Copilot 指示

Copilot 支援在儲存庫中透過 `.github/skills`、`.claude/skills` 或 `.agents/skills` 目錄建立 agent skills。

```bash
mkdir -p .github

# 建立必要 skills 的檔案
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .github/skills/test-driven-development/SKILL.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md > .github/skills/code-review-and-quality/SKILL.md
```

更多細節請參考 [Creating agent skills for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-skills)。

### Agent Personas（agents.md）

Copilot 支援專門化的 agent persona。可直接使用 agent-skills 內建的 agents：

```bash
# 複製 agent 定義
cp /path/to/agent-skills/agents/code-reviewer.md .github/agents/code-reviewer.md
cp /path/to/agent-skills/agents/test-engineer.md .github/agents/test-engineer.md
cp /path/to/agent-skills/agents/security-auditor.md .github/agents/security-auditor.md
```

在 Copilot Chat 中呼叫 agents：
- `@code-reviewer Review this PR`
- `@test-engineer Analyze test coverage for this module`
- `@security-auditor Check this endpoint for vulnerabilities`

### 自訂指示（使用者層級）

如果你希望某些 skills 套用到所有儲存庫：

1. 開啟 VS Code → Settings → GitHub Copilot → Custom Instructions
2. 加入你最常使用的 skill 摘要

## 建議設定

### .github/copilot-instructions.md

GitHub Copilot 支援透過 `.github/copilot-instructions.md` 提供專案層級指示。

```markdown
# 專案程式碼標準

## 測試
- 先寫測試再寫程式碼（TDD）
- 修 bug 時：先寫一個失敗的測試，再修正（Prove-It pattern）
- 測試層級：unit > integration > e2e（用能捕捉行為的最低層級）
- 每次變更後都執行 `npm test`

## 程式碼品質
- 從五個面向審查：正確性、可讀性、架構、安全性、效能
- 每個 PR 都必須通過：lint、type check、tests、build
- 程式碼與版本控制中不得包含任何 secrets

## 實作
- 以小且可驗證的增量方式建構
- 每個增量：實作 → 測試 → 驗證 → 提交
- 不要把格式化變更和行為變更混在一起

## 邊界
- 一律要做：提交前執行測試、驗證使用者輸入
- 先詢問：資料庫 schema 變更、新相依套件
- 絕對不要：提交 secrets、移除失敗中的測試、跳過驗證
```

### 專門化 Agents

在 Copilot Chat 中，使用這些 agents 來進行針對性的審查流程。

## 使用秘訣

1. **保持指示精簡** — Copilot 的指示在聚焦時效果最好。請摘要關鍵規則，不要把完整 skill 檔案全部貼上。
2. **審查時使用 agents** — `code-reviewer`、`test-engineer` 與 `security-auditor` 這些 agents 是為 Copilot 的 agent 模型設計的。
3. **在聊天中引用** — 當你處於特定階段工作時，可把對應 skill 內容貼進 Copilot Chat，補足上下文。
4. **搭配 PR reviews** — 設定 Copilot 在審查 PR 時使用 `code-reviewer` agent persona。
