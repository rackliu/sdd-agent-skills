# 在 Cursor 中使用 agent-skills

## 設定

### 方案 1：Rules 目錄（建議）

Cursor 支援使用 `.cursor/rules/` 目錄來放置專案專屬規則：

```bash
# 建立 rules 目錄
mkdir -p .cursor/rules

# 複製要作為規則使用的 skills
cp /path/to/agent-skills/skills/test-driven-development/SKILL.md .cursor/rules/test-driven-development.md
cp /path/to/agent-skills/skills/code-review-and-quality/SKILL.md .cursor/rules/code-review-and-quality.md
cp /path/to/agent-skills/skills/incremental-implementation/SKILL.md .cursor/rules/incremental-implementation.md
```

此目錄中的規則會自動載入到 Cursor 的上下文中。

### 方案 2：.cursorrules 檔案

在專案根目錄建立 `.cursorrules` 檔案，並將必要 skills 內嵌進去：

```bash
# 產生合併後的 rules 檔案
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .cursorrules
echo "\n---\n" >> .cursorrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .cursorrules
```

### 方案 3：Notepads

Cursor 的 Notepads 功能可讓你儲存可重複使用的上下文。可為每個常用 skill 建立一個 notepad：

1. 開啟 Cursor → Settings → Notepads
2. 建立一個新的 notepad，名稱為 "swe: Test-Driven Development"
3. 貼上 `skills/test-driven-development/SKILL.md` 的內容
4. 在聊天中用 `@notepad swe: Test-Driven Development` 引用它

## 建議設定

### 必要 Skills（永遠載入）

將下列內容加入 `.cursor/rules/`：

1. `test-driven-development.md` — TDD 工作流程與 Prove-It pattern
2. `code-review-and-quality.md` — 五面向審查
3. `incremental-implementation.md` — 以小而可驗證的切片方式建構

### 階段型 Skills（以 Notepads 載入）

為情境式使用的 skills 建立 notepads：

- "swe: Spec Development" → `spec-driven-development/SKILL.md`
- "swe: Frontend UI" → `frontend-ui-engineering/SKILL.md`
- "swe: Security" → `security-and-hardening/SKILL.md`
- "swe: Performance" → `performance-optimization/SKILL.md`

在處理相關任務時，用 `@notepad` 引用它們。

## 使用秘訣

1. **不要一次載入所有 skills** — Cursor 有上下文限制。建議只把 2 到 3 個 skill 當成 rules，其餘放在 notepads。
2. **明確引用 skills** — 直接告訴 Cursor「請依照 test-driven-development 規則處理這次變更」，確保它真的會讀取已載入的規則。
3. **審查時使用 agents** — 複製 `agents/code-reviewer.md` 的內容，並告訴 Cursor「請用這套程式碼審查框架檢視這份 diff」。
4. **按需載入 references** — 需要處理效能時，可引用 `@notepad performance-checklist`，或直接貼上 checklist 內容。
