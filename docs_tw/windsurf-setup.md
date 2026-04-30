# 在 Windsurf 中使用 agent-skills

## 設定

### 專案規則

Windsurf 使用 `.windsurfrules` 來放置專案專屬的 agent 指示：

```bash
# 從你最重要的 skills 建立一份合併後的規則檔案
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md >> .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .windsurfrules
```

### 全域規則

如果你希望某些 skills 套用到所有專案，可將它們加入 Windsurf 的全域規則：

1. 開啟 Windsurf → Settings → AI → Global Rules
2. 貼上你最常使用的 skills 內容

## 建議設定

讓 `.windsurfrules` 聚焦於 2 到 3 個核心 skills，避免超出上下文限制：

```
# .windsurfrules
# 本專案的核心 agent-skills

[貼上 test-driven-development SKILL.md]

---

[貼上 incremental-implementation SKILL.md]

---

[貼上 code-review-and-quality SKILL.md]
```

## 使用秘訣

1. **有選擇地載入** — Windsurf 的上下文有限。請優先選擇最能補足你主要品質缺口的 skills。
2. **在對話中引用** — 當你處於特定階段時，可把額外 skill 內容貼進聊天中，例如在做 auth 時貼上 `security-and-hardening`。
3. **把 references 當作 checklist 使用** — 貼上 `references/security-checklist.md`，並要求 Windsurf 逐項驗證。
