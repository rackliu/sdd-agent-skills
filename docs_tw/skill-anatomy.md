# Skill 結構說明

本文件說明 agent-skills 中 skill 檔案的結構與格式。當你要貢獻新 skill，或理解既有 skill 時，可把它當作參考指南。

## 檔案位置

每個 skill 都位於 `skills/` 下的獨立目錄中：

```
skills/
  skill-name/
    SKILL.md           # 必填：skill 定義
    supporting-file.md # 選填：按需載入的參考資料
```

## SKILL.md 格式

### Frontmatter（必填）

```yaml
---
name: skill-name-with-hyphens
description: Guides agents through [task/workflow]. Use when [specific trigger conditions].
---
```

**規則：**
- `name`：小寫、以連字號分隔，且必須與目錄名稱一致。
- `description`：先用第三人稱描述這個 skill 做什麼，再加入一個或多個清楚的「Use when」觸發條件。必須同時說明 *做什麼* 與 *何時使用*。上限 1024 字元。

**為什麼這很重要：** Agents 會透過 description 來探索 skills。這段描述會被注入 system prompt，因此它必須同時告訴 agent 這個 skill 提供什麼能力，以及何時應該啟用。不要把工作流程摘要直接寫進 description；如果 description 裡出現流程步驟，agent 可能只照著摘要做，而不去讀完整 skill。

### 標準段落（建議模式）

```markdown
# Skill Title

## Overview
用一到兩句說明這個 skill 做什麼，以及它的重要性。

## When to Use
- 觸發條件清單（症狀、任務類型）
- 不該使用的情境（排除項）

## [Core Process / The Workflow / Steps]
主要工作流程，拆成編號步驟或階段。
需要時加入程式碼範例。
若有決策分歧，可使用流程圖（ASCII）。

## [Specific Techniques / Patterns]
針對特定情境的細部指引。
包含程式碼範例、範本、設定。

## Common Rationalizations
| Rationalization | Reality |
|---|---|
| agents 用來跳過步驟的藉口 | 為什麼這個藉口是錯的 |

## Red Flags
- 表示此 skill 正被違反的行為模式
- 審查時應特別留意的事項

## Verification
完成 skill 流程後，請確認：
- [ ] 完成條件檢查清單
- [ ] 證據要求
```

## 各段落用途

### Overview
這是 skill 的「電梯簡報版」說明。應回答：這個 skill 在做什麼？為什麼 agent 應該遵循它？

### When to Use
幫助 agents 與人類判斷這個 skill 是否適用於目前任務。要同時包含正向觸發條件（「Use when X」）與排除條件（「NOT for Y」）。

### Core Process
這是 skill 的核心，也是 agent 要遵循的 step-by-step 工作流程。內容必須具體且可執行，不能只是模糊建議。

**好範例：** 「執行 `npm test` 並確認所有測試通過」
**壞範例：** 「確認測試有正常運作」

### Common Rationalizations
這是高品質 skill 最有辨識度的部分。這裡列出 agents 常用來跳過重要步驟的藉口，並附上對應反駁。它的作用是避免 agent 用自我合理化的方式逃避流程。

想想看 agent 曾說過的那些話，例如「我之後再補測試」或「這個需求太簡單，不需要 spec」；這些都應該寫進這張表，並附上事實性的反駁理由。

### Red Flags
可觀察到的訊號，表示 skill 可能沒有被正確遵守。這在程式碼審查與自我檢查時都很有用。

### Verification
完成條件。agent 會用這份 checklist 確認 skill 流程是否已完成。每一個勾選項目都應該能以證據驗證（例如測試輸出、build 結果、截圖等）。

## Supporting Files

只有在以下情況才建立 supporting files：
- 參考資料超過 100 行（讓主要的 SKILL.md 保持聚焦）
- 需要程式工具或腳本
- Checklist 長到值得拆成獨立檔案

如果模式與原則少於 50 行，請盡量直接內嵌在主檔中。

## 撰寫原則

1. **流程優先於知識。** Skills 是工作流程，不是參考文件。寫步驟，不要只寫事實。
2. **具體優先於籠統。** 「執行 `npm test`」比「驗證測試」更好。
3. **證據優先於假設。** 每個 verification 勾選項都必須有證據支持。
4. **反合理化。** 每一個容易被跳過的步驟，都應在 rationalizations 表中提供反駁理由。
5. **漸進揭露。** 主 SKILL.md 是入口；supporting files 只在需要時載入。
6. **注意 token 成本。** 每個段落都必須值得存在。如果刪掉後不會改變 agent 行為，就應該刪除。

## 命名慣例

- Skill 目錄：`lowercase-hyphen-separated`
- Skill 檔案：`SKILL.md`（永遠大寫）
- Supporting files：`lowercase-hyphen-separated.md`
- References：放在專案根目錄的 `references/`，而不是 skill 目錄內

## Cross-Skill References

以名稱引用其他 skills：

```markdown
Follow the `test-driven-development` skill for writing tests.
If the build breaks, use the `debugging-and-error-recovery` skill.
```

不要在 skills 之間重複內容，應改用引用與連結。
