# SDD Agent Skills 軟體開發導入手冊

## 使用 `agent-skills` 進行系統化 AI 輔助軟體開發

**版本：** 1.0  
**適用對象：** 技術主管、資深工程師、帶領 AI 輔助開發團隊的 Scrum Master

---

## 前言

### 為什麼 AI 輔助開發需要 `agent-skills`？

AI 編程代理（Claude Code、GitHub Copilot、Cursor、Gemini CLI 等）的能力已相當強大，但若缺乏明確的工程紀律約束，它們常見的失敗模式包括：

- **跳過規格直接寫程式碼** — 在需求模糊時憑假設動手，導致方向偏離
- **一次寫完所有程式** — 難以審查、測試困難、難以回滾
- **先寫程式再補測試** — 測試變成形式，失去防護功能
- **忽略安全性考量** — OWASP Top 10 漏洞悄悄進入生產環境
- **推進時遇到錯誤不停下** — 累積錯誤，後期修復成本呈指數成長
- **維護期破壞現有功能** — 新功能上線卻弄壞舊功能（回歸問題）

`agent-skills` 的解決方案是：**將資深工程師的工作紀律打包成 AI 代理可直接遵循的結構化流程（Skills）**。每個 Skill 包含：

- **明確的觸發條件** — 什麼情況下必須啟用這個技能
- **逐步的工作流程** — 代理必須按順序執行，不得跳過
- **驗證閘門（Verification Gates）** — 在進入下一步前，必須通過人工確認或自動化測試
- **反合理化機制** — 列出代理可能用來跳過步驟的藉口，以及正確的反駁
- **完成標準（Exit Criteria）** — 明確定義「什麼叫做完成」

### 三個核心組成層次

`agent-skills` 由三個可組合的層次構成，各司其職：

| 層次 | 位置 | 職責 | 範例 |
|------|------|------|------|
| **技能（Skills）** | `skills/<name>/SKILL.md` | 工作流程 — *如何做* | `spec-driven-development`、`test-driven-development` |
| **角色（Agents/Personas）** | `agents/<role>.md` | 專家視角 — *由誰做* | `code-reviewer`、`security-auditor` |
| **指令（Commands）** | `.claude/commands/*.md` | 進入點 — *何時觸發* | `/spec`、`/review`、`/ship` |

**組合規則：** 使用者（或 Slash Command）是指揮者。角色不呼叫其他角色；技能由角色或指令從內部調用。唯一認可的多角色協作模式是 `/ship` 的**平行扇出**（同時觸發 `code-reviewer`、`security-auditor`、`test-engineer`，再彙整報告）。

### 開發生命週期總覽

六個開發階段，對應 7 個 Slash Command，底層由 20 個技能驅動：

```
┌──────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│  DEFINE  │   PLAN   │  BUILD   │  VERIFY  │  REVIEW  │   SHIP   │
│  定義    │  規劃    │  建構    │  驗證    │  審查    │  上線    │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ /spec    │ /plan    │ /build   │ /test    │ /review  │ /ship    │
│          │          │          │          │/simplify │          │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ idea-    │planning- │incremen- │debugging-│code-     │git-      │
│ refine   │and-task- │tal-impl  │and-error │review-   │workflow  │
│ spec-    │breakdown │test-     │-recovery │and-      │ci-cd-    │
│ driven   │          │driven-   │browser-  │quality   │automati  │
│          │          │develop   │testing   │security  │shipping- │
│          │          │frontend  │          │          │and-lauch │
└──────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```

### 全部 20 個技能一覽

| 階段 | 技能名稱 | 核心功能 |
|------|---------|---------|
| **Define** | `idea-refine` | 發散/收斂思考，將模糊想法具體化 |
| **Define** | `spec-driven-development` | 先寫 PRD 規格，再寫程式碼 |
| **Plan** | `planning-and-task-breakdown` | 依賴圖分析，拆解為可驗證的小任務 |
| **Build** | `incremental-implementation` | 薄片垂直切片：實作→測試→驗證→提交 |
| **Build** | `test-driven-development` | Red-Green-Refactor，測試金字塔 80/15/5 |
| **Build** | `context-engineering` | 在正確時機向代理提供正確的上下文資訊 |
| **Build** | `source-driven-development` | 所有框架決策基於官方文件，可引用來源 |
| **Build** | `frontend-ui-engineering` | 元件架構、設計系統、WCAG 2.1 無障礙設計 |
| **Build** | `api-and-interface-design` | 合約優先設計、Hyrum 定律、邊界驗證 |
| **Verify** | `browser-testing-with-devtools` | Chrome DevTools MCP 即時 DOM/網路/效能偵錯 |
| **Verify** | `debugging-and-error-recovery` | 五步驟分類：重現→定位→簡化→修復→防護 |
| **Review** | `code-review-and-quality` | 五軸審查（正確性/可讀性/架構/安全性/效能） |
| **Review** | `code-simplification` | Chesterton 柵欄原則，保持行為不變下簡化 |
| **Review** | `security-and-hardening` | OWASP Top 10、認證模式、機密管理、三層邊界 |
| **Review** | `performance-optimization` | 先量測再優化，Core Web Vitals 目標 |
| **Ship** | `git-workflow-and-versioning` | 主幹開發、原子式提交，約 100 行/次變更 |
| **Ship** | `ci-cd-and-automation` | 左移原則、功能旗標、品質閘門流水線 |
| **Ship** | `deprecation-and-migration` | 廢棄程式碼管理、遷移模式、Feature Flag 退場 |
| **Ship** | `documentation-and-adrs` | 架構決策記錄（ADR）、API 文件標準 |
| **Ship** | `shipping-and-launch` | 上線前 Checklist、分段推出、回滾程序 |

### 三個專家角色

可在需要針對性審查時載入對應角色：

| 角色 | 文件 | 適用情境 |
|------|------|---------|
| `code-reviewer` | `agents/code-reviewer.md` | PR 合併前，Staff Engineer 標準的五軸審查 |
| `test-engineer` | `agents/test-engineer.md` | 測試策略設計、覆蓋率分析、Prove-It 模式 |
| `security-auditor` | `agents/security-auditor.md` | 弱點偵測、威脅建模、OWASP 完整評估 |

---

## 專案架構

```
agent-skills/
├── skills/                    # 20 個技能定義
│   ├── spec-driven-development/
│   │   └── SKILL.md           # 每個技能的完整工作流程文件
│   ├── test-driven-development/
│   │   └── SKILL.md
│   ├── idea-refine/
│   │   ├── SKILL.md
│   │   ├── examples.md        # 部分技能含輔助文件
│   │   └── scripts/
│   │       └── idea-refine.sh
│   └── ... (共 20 個技能目錄)
│
├── agents/                    # 3 個專家角色 Persona
│   ├── code-reviewer.md
│   ├── test-engineer.md
│   ├── security-auditor.md
│   └── README.md              # Persona 使用決策矩陣
│
├── .claude/
│   └── commands/              # 7 個 Slash Command 定義
│       ├── spec.md            # /spec → spec-driven-development
│       ├── plan.md            # /plan → planning-and-task-breakdown
│       ├── build.md           # /build → incremental-implementation + TDD
│       ├── test.md            # /test → debugging-and-error-recovery
│       ├── review.md          # /review → code-review-and-quality
│       ├── code-simplify.md   # /code-simplify → code-simplification
│       └── ship.md            # /ship → 平行扇出三個角色
│
├── references/                # 可重用品質檢查清單
│   ├── accessibility-checklist.md
│   ├── security-checklist.md
│   ├── performance-checklist.md
│   ├── testing-patterns.md
│   └── orchestration-patterns.md
│
├── hooks/                     # 工作階段生命週期鉤子
│   ├── hooks.json             # 鉤子設定
│   ├── session-start.sh       # 會話開始時執行
│   └── sdd-cache-pre/post.sh  # 規格快取機制
│
└── docs/                      # 各 AI 工具的安裝指南
    ├── getting-started.md     # 通用安裝說明
    ├── copilot-setup.md       # GitHub Copilot
    ├── cursor-setup.md        # Cursor
    ├── gemini-cli-setup.md    # Gemini CLI
    ├── windsurf-setup.md      # Windsurf
    ├── opencode-setup.md      # OpenCode
    └── skill-anatomy.md       # 如何撰寫新技能
```

### SKILL.md 標準結構

每個技能文件都遵循相同的結構，確保代理可以可預期地解讀與執行：

```
─── YAML Frontmatter ────────────────────────────────
name: skill-name           # 與目錄名稱一致
description: 說明何時啟用  # 代理用此判斷是否適用

─── 文件內容 ────────────────────────────────────────
## Overview          → 這個技能做什麼、為什麼重要
## When to Use       → 觸發條件（正向/負向）
## Core Process      → 逐步工作流程（核心，不可跳過）
## Common Rationalizations → 代理跳過步驟的藉口 + 反駁
## Red Flags         → 技能被違反的警告信號
## Verification      → 完成標準 Checklist
```

---

## 安裝與設定

### 方法一：Claude Code（最完整，推薦）

Claude Code 支援 Plugin 系統，技能可自動依情境觸發。

**從 Marketplace 安裝：**
```bash
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

**本地開發安裝：**
```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

安裝後，在 Claude Code 對話中直接使用 Slash Command：
```
/spec    → 開始撰寫規格（需求不明確時）
/plan    → 拆解任務（有規格但不知如何下手）
/build   → 逐片實作（開始寫程式碼）
/test    → 偵錯（測試失敗、行為異常）
/review  → 審查（合併前）
/ship    → 上線準備（部署前）
```

---

### 方法二：GitHub Copilot（VS Code / JetBrains）

**步驟 1：複製技能文件**
```bash
mkdir -p .github/skills

# 複製核心技能（依需求選擇）
cp /path/to/agent-skills/skills/spec-driven-development/SKILL.md \
   .github/skills/spec-driven-development/SKILL.md
cp /path/to/agent-skills/skills/test-driven-development/SKILL.md \
   .github/skills/test-driven-development/SKILL.md
cp /path/to/agent-skills/skills/code-review-and-quality/SKILL.md \
   .github/skills/code-review-and-quality/SKILL.md
```

**步驟 2：設定 Copilot 專案指令**

建立 `.github/copilot-instructions.md`：
```markdown
# 專案開發規範

## 測試
- 先寫測試再寫程式碼（TDD）
- 修 Bug 時：先寫失敗的測試，再修復
- 執行 `npm test` 確認每次變更後測試通過

## 品質
- 合併前執行五軸審查（正確性/可讀性/架構/安全性/效能）
- 每個 PR 必須通過：lint、型別檢查、測試、建構

## 實作
- 薄片垂直切片，每次約 100 行變更
- 循環：實作 → 測試 → 驗證 → 提交

## 邊界
- 必做：提交前執行測試、驗證使用者輸入
- 詢問後再做：資料庫 Schema 異動、新增相依套件
- 絕不：提交金鑰、刪除失敗測試、跳過驗證
```

**步驟 3：啟用專家角色**
```bash
mkdir -p .github/agents
cp /path/to/agent-skills/agents/code-reviewer.md .github/agents/
cp /path/to/agent-skills/agents/test-engineer.md .github/agents/
cp /path/to/agent-skills/agents/security-auditor.md .github/agents/
```

在 Copilot Chat 中呼叫角色：
```
@code-reviewer   Review this PR for me
@test-engineer   Analyze test coverage for the sales module
@security-auditor Check this API endpoint for vulnerabilities
```

---

### 方法三：Cursor

**選項 A：規則目錄（推薦）**
```bash
mkdir -p .cursor/rules
cp /path/to/agent-skills/skills/test-driven-development/SKILL.md \
   .cursor/rules/test-driven-development.md
cp /path/to/agent-skills/skills/incremental-implementation/SKILL.md \
   .cursor/rules/incremental-implementation.md
cp /path/to/agent-skills/skills/code-review-and-quality/SKILL.md \
   .cursor/rules/code-review-and-quality.md
```

`.cursor/rules/` 目錄內的文件會自動載入到 Cursor 的上下文中。

**選項 B：Notepads（情境式技能）**
1. 開啟 Cursor → Settings → Notepads
2. 建立 Notepad，命名為 `swe: Spec Development`
3. 貼上 `skills/spec-driven-development/SKILL.md` 的內容
4. 對話中使用 `@notepad swe: Spec Development` 呼叫

---

### 方法四：Gemini CLI

**原生技能安裝（推薦）：**
```bash
# 從 GitHub 安裝
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills

# 或從本地複製安裝
git clone https://github.com/addyosmani/agent-skills.git
gemini skills install ./agent-skills/skills/

# 確認安裝成功
/skills list
```

Gemini CLI 會自動依任務類型觸發對應技能，使用前會詢問是否啟用。

**持久化設定（GEMINI.md）：**
```bash
# 常駐技能寫入 GEMINI.md（每次對話自動載入）
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md > GEMINI.md
echo -e "\n---\n" >> GEMINI.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> GEMINI.md
```

---

### 方法五：Windsurf

**建立 `.windsurfrules` 文件：**
```bash
# 合併核心技能
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .windsurfrules
echo -e "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md >> .windsurfrules
echo -e "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .windsurfrules
```

---

### 方法六：任何其他 AI 代理（通用方式）

技能文件是純 Markdown，可用於任何接受系統提示或指令文件的 AI 代理：

```bash
# 查看技能內容
cat skills/spec-driven-development/SKILL.md

# 直接貼入：
# - 對話開頭（系統提示）
# - 專案的規則文件（CLAUDE.md、.cursorrules 等）
# - 任務說明的開頭
```

也可以在對話中明確指示代理：
```
「請依照 test-driven-development 流程實作這個功能，
 先寫失敗的測試，再寫最小程式碼使其通過，最後重構。」
```

---

### 建議的最小配置（新團隊起步）

若不確定從哪裡開始，以下三個技能涵蓋最關鍵的品質防護：

| 技能 | 解決的問題 |
|------|-----------|
| `spec-driven-development` | 防止代理在需求模糊時擅自猜測實作 |
| `test-driven-development` | 確保測試是設計工具，不是事後補充 |
| `code-review-and-quality` | 提供一致的合併前品質標準 |

---

## 範例一：新專案開發 — 服裝飾品店銷售管理系統

### 背景

某中型服裝飾品連鎖店（4 家門市）需要建立一套銷售管理系統，取代目前以 Excel 管理的流程。技術團隊決定導入 `agent-skills` 進行 AI 輔助開發。

---

### Phase 1：DEFINE — 定義要建構什麼

**對應技能：** `idea-refine` → `spec-driven-development`  
**對應指令：** `/spec`

#### 步驟 1.1：想法精煉（idea-refine）

工程師對 AI 代理發出需求（此時技能 `idea-refine` 已載入）：

```
使用者：
我想要建立一個服裝飾品店的銷售管理系統，
目前用 Excel 管理，想要改成線上系統。
```

**代理（套用 `idea-refine` 技能後的行為）：**

代理會引導發散性思考，主動提出澄清問題：

```
代理回應：
在開始規格撰寫之前，我需要確認幾個方向：

【業務維度】
1. 現在 Excel 最大的痛點是什麼？（庫存算錯？報表費時？多人同時編輯衝突？）
2. 4 家門市是否需要即時共享庫存？還是每家獨立管理？
3. 是否有線上銷售（官網 / 蝦皮）需要整合？

【使用者維度】
4. 實際使用者角色有哪些？（收銀員、店長、總管理員？）
5. 行動裝置使用需求？（店員需要用手機查庫存嗎？）

【技術維度】
6. 現有 IT 基礎設施？（自建伺服器還是採用雲端？）
7. 是否需要對接現有 ERP / 會計系統？

請先回答以上問題，我再進行規格撰寫。
```

**客戶回應後，代理整理出具體提案：**

```
精煉後的核心提案：
1. 多門市共享庫存的 Web 系統（雲端部署）
2. 三種角色：收銀員 / 店長 / 系統管理員
3. 核心功能：POS 銷售、庫存管理、促銷折扣、多門市報表
4. 需支援手機瀏覽器（RWD）
5. 第一階段不對接 ERP，預留介面
```

---

#### 步驟 1.2：規格撰寫（spec-driven-development）

代理（套用 `spec-driven-development` 技能）**在寫任何程式碼之前**，先明列假設：

```
我將基於以下假設撰寫規格，請確認後我再繼續：

ASSUMPTIONS：
1. 前端框架：Next.js 14（App Router）
2. 後端 API：Next.js API Routes + Prisma ORM
3. 資料庫：PostgreSQL（Supabase 雲端託管）
4. 認證：NextAuth.js，角色型存取控制（RBAC）
5. 部署：Vercel（前端）+ Supabase（資料庫）
6. 測試：Jest + React Testing Library + Playwright（E2E）

→ 請確認或修正後，我再繼續撰寫規格文件。
```

**確認後，代理產出的規格文件（PRD）涵蓋六個核心領域：**

```markdown
# 服裝飾品店銷售管理系統 - 規格文件 v1.0

## 1. Objective（目標）
建立一套多門市共用的 Web 銷售管理系統，解決：
- 跨門市庫存即時同步
- 銷售報表自動化（取代每週手動整理 Excel）
- 角色型存取控制（確保收銀員僅能操作 POS）

成功指標：
- 月結報表生成時間從 4 小時降至 5 分鐘
- 庫存盤點差異率 < 0.5%

## 2. Commands（可執行指令）
Build:   npm run build
Test:    npm test -- --coverage --coverageThreshold='{"global":{"lines":80}}'
Lint:    npm run lint
Dev:     npm run dev
E2E:     npx playwright test
Migrate: npx prisma migrate deploy
Seed:    npx prisma db seed

## 3. Structure（系統結構）
apps/
  web/                     # Next.js 前端
    app/
      (auth)/login/        # 登入頁
      (dashboard)/         # 主控台（需認證）
        pos/               # POS 銷售介面
        inventory/         # 庫存管理
        reports/           # 報表
        admin/             # 系統管理（僅限 ADMIN）
    components/
    lib/
  api/                     # API Routes（內嵌在 Next.js）
packages/
  db/                      # Prisma schema + migrations
  types/                   # 共用 TypeScript 型別

## 4. Code Style（程式碼風格）
- TypeScript strict mode（不允許 any）
- 元件 Props 必須定義 interface
- 資料庫操作統一透過 Repository Pattern
- 禁止在元件內直接呼叫 Prisma

## 5. Testing（測試策略）
- Unit Tests（80%）：業務邏輯、計算函式、Repository
- Integration Tests（15%）：API Routes、資料庫操作
- E2E Tests（5%）：完整銷售流程、登入登出、報表生成

## 6. Boundaries（邊界與禁止事項）
- 不實作：行動原生 App（RWD 取代）
- 不實作：ERP 整合（第一階段，預留 API 介面）
- 不實作：電商整合（線路外）
- 禁止：硬編碼任何金鑰或密碼至程式碼
- 禁止：在 API 未驗證角色前提下存取資料
```

> **重要閘門：** 代理在此等待人工確認規格，**不得提前進入 PLAN 階段**。

---

### Phase 2：PLAN — 拆解任務

**對應技能：** `planning-and-task-breakdown`  
**對應指令：** `/plan`

代理進入「唯讀計劃模式」，**不寫任何程式碼**，只輸出計劃文件：

#### 依賴圖分析

```
資料庫 Schema（Prisma）
    │
    ├── 型別定義（TypeScript types）
    │       │
    │       ├── Repository 層（資料存取）
    │       │       │
    │       │       ├── API Routes（Next.js）
    │       │       │       │
    │       │       │       └── 前端 API Client（fetch hooks）
    │       │       │               │
    │       │       │               └── UI 元件（React）
    │       │
    │       └── 驗證邏輯（Zod schemas）
    │
    └── Seed 資料 / Migrations
```

#### 任務清單（依實作優先順序）

```
Sprint 1 — 基礎骨架（2 週）
─────────────────────────────
Task 1.1  資料庫 Schema 設計（Product, Inventory, Sale, User, Store）
Task 1.2  Prisma Migration + Seed 資料（測試用商品 50 筆）
Task 1.3  NextAuth.js 認證設定（Email/Password + RBAC）
Task 1.4  登入 / 登出 UI（含角色導向重定向）
Task 1.5  CI 管道設定（GitHub Actions：lint → test → build）

驗收標準：
- [ ] 所有 Migration 可無錯誤執行
- [ ] 三種角色可成功登入並被導向正確頁面
- [ ] CI 管道自動觸發並通過

Sprint 2 — POS 銷售模組（2 週）
─────────────────────────────────
Task 2.1  商品搜尋 API（支援條碼 / 關鍵字）
Task 2.2  購物車狀態管理（useCart hook）
Task 2.3  POS 收銀 UI（商品列表 + 購物車 + 結帳）
Task 2.4  銷售交易 API（建立 Sale + 扣減庫存，DB Transaction）
Task 2.5  收據列印（瀏覽器 Print API）

驗收標準：
- [ ] 收銀員可完成完整銷售流程（測試：unit + E2E）
- [ ] 庫存在銷售後正確扣減
- [ ] 並發銷售不造成庫存超扣（Transaction 測試）

Sprint 3 — 庫存管理模組（2 週）
─────────────────────────────────
Task 3.1  庫存查詢 API（含門市篩選）
Task 3.2  庫存調整 API（進貨 / 調撥 / 盤點）
Task 3.3  庫存管理 UI（表格 + 篩選 + 匯出 CSV）
Task 3.4  低庫存警示（閾值設定 + 儀表板通知）

Sprint 4 — 報表與管理（2 週）
─────────────────────────────
Task 4.1  銷售報表 API（日 / 週 / 月，可依門市篩選）
Task 4.2  報表 UI（圖表：recharts + 表格匯出）
Task 4.3  使用者管理（CRUD，限 ADMIN）
Task 4.4  商品管理（CRUD + 圖片上傳）
Task 4.5  系統設定（門市資料、促銷規則）

Sprint 5 — 測試強化與上線準備（1 週）
──────────────────────────────────────
Task 5.1  E2E 測試補全（Playwright，完整業務流程）
Task 5.2  效能優化（Lighthouse 稽核、N+1 查詢修正）
Task 5.3  安全性稽核（OWASP Top 10 逐項確認）
Task 5.4  上線前 Checklist（shipping-and-launch 技能）
```

> **重要閘門：** 人工確認任務清單後，才進入 BUILD 階段。

---

### Phase 3：BUILD — 逐片實作

**對應技能：** `incremental-implementation` + `test-driven-development`  
**對應指令：** `/build`

每一個 Task 都遵循相同的**實作循環**：

```
實作循環（以 Task 2.4 銷售交易 API 為例）
────────────────────────────────────────────

Step 1：先寫失敗的測試（Red）
─────────────────────────────
// __tests__/api/sales.test.ts
describe('POST /api/sales', () => {
  it('建立銷售並扣減庫存', async () => {
    // Arrange
    await seedProduct({ id: 'p1', stock: 10 });
    
    // Act
    const res = await POST('/api/sales', {
      storeId: 'store1',
      items: [{ productId: 'p1', quantity: 3, price: 299 }]
    });
    
    // Assert
    expect(res.status).toBe(201);
    const product = await db.product.findUnique({ where: { id: 'p1' } });
    expect(product.stock).toBe(7);  // 10 - 3 = 7
  });

  it('並發銷售不造成庫存超扣', async () => {
    await seedProduct({ id: 'p2', stock: 5 });
    
    // 同時發出 6 筆訂單，每筆購買 1 件（總需求 6，庫存只有 5）
    const results = await Promise.all(
      Array.from({ length: 6 }, () =>
        POST('/api/sales', {
          items: [{ productId: 'p2', quantity: 1, price: 100 }]
        })
      )
    );
    
    const successCount = results.filter(r => r.status === 201).length;
    expect(successCount).toBe(5);    // 最多成功 5 筆
    
    const product = await db.product.findUnique({ where: { id: 'p2' } });
    expect(product.stock).toBe(0);   // 庫存恰好用完，不為負數
  });
});

// 執行測試 → 預期失敗（API 尚未實作）

Step 2：實作最小程式碼使測試通過（Green）
──────────────────────────────────────────
// app/api/sales/route.ts
export async function POST(request: Request) {
  const session = await getServerSession(authOptions);
  if (!session) return Response.json({ error: 'Unauthorized' }, { status: 401 });
  
  const body = await request.json();
  const parsed = CreateSaleSchema.safeParse(body);
  if (!parsed.success) {
    return Response.json({ error: parsed.error }, { status: 400 });
  }
  
  try {
    const sale = await db.$transaction(async (tx) => {
      // 在 Transaction 內鎖定庫存，防止超扣
      for (const item of parsed.data.items) {
        const product = await tx.product.findUnique({
          where: { id: item.productId },
          select: { stock: true }
        });
        
        if (!product || product.stock < item.quantity) {
          throw new Error(`庫存不足：${item.productId}`);
        }
        
        await tx.product.update({
          where: { id: item.productId },
          data: { stock: { decrement: item.quantity } }
        });
      }
      
      return tx.sale.create({ data: { ...parsed.data } });
    });
    
    return Response.json(sale, { status: 201 });
  } catch (error) {
    if (error.message.includes('庫存不足')) {
      return Response.json({ error: error.message }, { status: 409 });
    }
    throw error;
  }
}

// 執行測試 → 預期通過

Step 3：重構（Refactor）
──────────────────────────
// 將業務邏輯提取到 SaleRepository，保持 API Route 清潔
// 確保重構後測試仍通過

Step 4：提交
────────────
git commit -m "feat(sales): 新增銷售 API，含庫存扣減 Transaction 保護

- POST /api/sales：建立銷售並原子性扣減庫存
- 並發保護：Transaction 防止庫存超扣
- 驗證：Zod schema 驗證輸入
- 測試：unit + 並發情境測試

Closes #T2.4"
```

> **技能強制規則：** 代理**不得在測試失敗時提交**，也**不得跳過 Red 步驟**直接寫程式碼。

---

### Phase 4：VERIFY — 證明有效

**對應技能：** `debugging-and-error-recovery` + `browser-testing-with-devtools`  
**對應指令：** `/test`

假設在 Sprint 2 測試中發現問題：

```
情境：E2E 測試中，促銷折扣計算結果與預期不符
錯誤訊息：
  Expected: 269.10 (原價 299, 九折)
  Received: 269.09 (差 0.01)

代理（套用 debugging-and-error-recovery 技能）的行為：

Step 1：停止新增功能
  → 暫停 Task 2.5 收據列印的實作

Step 2：保存證據
  → 記錄：觸發條件（折扣 10%、原價 299）、環境（Node 20.x）

Step 3：重現問題
  → 撰寫最小重現程式碼：
     applyDiscount(299, 0.1)  // → 269.09（預期 269.10）

Step 4：定位根因
  → 發現：JavaScript 浮點數精度問題
     299 * 0.9 = 269.09000000000003（JS 浮點數）
     Math.round(299 * 0.9 * 100) / 100 = 269.09（捨入方向錯誤）

Step 5：修正根因
  → 改用整數運算：
     Math.round(price * (100 - discountPercent)) / 100

Step 6：防護（Guard）
  → 新增單元測試覆蓋邊界值：
     [299, 0.1] → 269.10
     [100, 0.3] → 70.00
     [1, 0.1]   → 0.90
  → 加入程式碼審查備註：「金額計算必須使用整數運算，禁止直接浮點乘除」
```

---

### Phase 5：REVIEW — 合併前品質閘門

**對應技能：** `code-review-and-quality` + `security-and-hardening`  
**對應指令：** `/review`

**每次 PR 合併前，代理執行五軸審查：**

```
PR：feat/sales-module（Sprint 2 完成，共 847 行變更）

代理審查報告：

【軸 1：正確性】✅
- 銷售流程涵蓋正常 / 庫存不足 / 未授權三種情境
- 浮點數計算已修正（#T2.4 bug 修正已含括）
- 邊界值測試：空購物車、數量為 0、價格為負數

【軸 2：可讀性】⚠️ FYI
- `useCart.ts` 第 89-134 行：狀態更新邏輯可提取為獨立函式
  （不阻擋合併，建議後續 Sprint 重構）

【軸 3：架構】⚠️ Optional
- SaleRepository 目前混合了業務驗證與資料存取邏輯
  建議：將庫存驗證提取到 InventoryService
  （不阻擋合併，但需在 Sprint 3 前處理）

【軸 4：安全性】🚨 Nit → 必須修正
- app/api/sales/route.ts 第 12 行：
  錯誤訊息直接回傳資料庫錯誤物件 `error.message`
  修正方式：
  // 錯誤的做法
  return Response.json({ error: error.message }, { status: 500 });
  // 正確的做法
  console.error('Sale creation failed:', error);  // 記錄完整錯誤到伺服器
  return Response.json({ error: '交易失敗，請稍後重試' }, { status: 500 });

【軸 5：效能】✅
- 查詢已正確使用索引（productId, storeId）
- 無 N+1 查詢問題

結論：安全性問題修正後可合併。
```

> **重要閘門：** 代理不得合併含有 `Nit` 等級以上安全問題的 PR。

---

### Phase 6：SHIP — 有信心地部署

**對應技能：** `shipping-and-launch` + `git-workflow-and-versioning`  
**對應指令：** `/ship`

Sprint 5 完成後，代理引導執行上線前檢查清單：

```
上線前檢查清單執行記錄
════════════════════════

【程式碼品質】
✅ npm test -- --coverage → 覆蓋率 84%（超過 80% 門檻）
✅ npm run build → 成功，無警告
✅ npm run lint → 0 錯誤
✅ PR 全數通過審查並合併
✅ 無未解決的 TODO（grep -r "TODO" 確認）

【安全性】
✅ npm audit → 0 critical, 0 high
✅ 所有 API 端點均有角色驗證（RBAC 稽核完成）
✅ 敏感資料已移至 .env（git log --all -S "password" 確認無洩漏）
✅ Rate limiting 設定（登入端點：15 次/分鐘）
✅ CORS 限制至正式網域

【效能】
✅ Lighthouse Score：Performance 91 / Accessibility 95 / Best Practices 100
✅ 最大商品列表查詢 < 200ms（P95）
✅ 資料庫索引：product.sku, sale.storeId, sale.createdAt

【部署策略（分段推出）】
Stage 1（第 1 天）：總公司管理員帳號 → 確認基本功能
Stage 2（第 2-3 天）：1 家門市（2 位收銀員）→ 觀察銷售流程
Stage 3（第 4-7 天）：2 家門市 → 確認多門市同步
Stage 4（第 8 天）：全部 4 家門市 → 完整上線

【回滾程序】
觸發條件：
- 錯誤率 > 1%（監控儀表板自動警示）
- 任何銷售資料異常

回滾步驟：
1. Vercel → 切換回前一個部署版本（< 2 分鐘）
2. 資料庫：Migration 皆可 rollback（npx prisma migrate reset 僅限緊急）
3. 通知所有門市暫停使用系統，改回 Excel 緊急模式

【監控設定】
✅ Sentry 錯誤追蹤（已設定 Slack 通知）
✅ Vercel Analytics（Core Web Vitals 即時監控）
✅ 資料庫連線池監控（Supabase Dashboard）
```

#### 最終上線結果

```
服裝飾品店銷售管理系統 v1.0 上線

日期：完成全部 5 個 Sprint（共 9 週）
結果：
  ✅ 4 家門市全數上線
  ✅ 月結報表從 4 小時 → 6 分鐘
  ✅ 庫存盤點差異率 0.2%（低於 0.5% 目標）
  ✅ 上線後 72 小時零重大事故
```

---
---

## 範例二：維護期需求異動管理

### 背景

服裝飾品店銷售管理系統已上線穩定運行 3 個月。業務部門提出新需求：

> **需求（口頭提出）：**
> 「我們想要加一個『會員點數』功能，消費可以累積點數，滿點可以折抵。」

---

### 維護期工作流程的差異

| 面向 | 新專案開發 | 維護期異動 |
|------|-----------|-----------|
| 規格起點 | 空白開始 | 基於現有系統 |
| 影響評估 | 不需要 | **必須評估對現有功能的衝擊** |
| 資料遷移 | 不需要 | 通常需要 Migration 舊資料 |
| 回歸風險 | 低 | 高（任何修改都可能破壞現有流程） |
| 廢棄管理 | 不需要 | 需要考慮舊功能的相容性 |

---

### Phase 1：DEFINE — 評估影響範圍

**對應技能：** `spec-driven-development`（維護模式）  
**對應指令：** `/spec`

#### 步驟 1.1：現有系統衝擊分析

代理**在撰寫規格前**，先進行現有程式碼的影響分析：

```
代理的衝擊分析報告：

1. 資料庫 Schema 影響
   需新增：Member 表、PointTransaction 表
   需修改：Sale 表（新增 memberId 欄位、pointsEarned、pointsRedeemed）
   → 需要資料庫 Migration（現有資料需相容）

2. API 影響
   需新增：/api/members（CRUD）
           /api/members/[id]/points（點數查詢）
   需修改：POST /api/sales（加入點數累積 / 折抵邏輯）
   → 現有 POS 整合測試需全面回歸

3. UI 影響
   需修改：POS 收銀介面（加入會員查詢、點數顯示、折抵選項）
   需新增：會員管理頁面（店長 / 管理員可見）
   → 需要 E2E 測試回歸現有銷售流程

4. 業務邏輯風險
   高風險：點數折抵與促銷折扣的**組合計算**邏輯複雜
   → 必須明確定義：「促銷折扣後再折抵點數」還是「先折抵再計算折扣」

ASSUMPTIONS（需確認）：
1. 點數規則：消費 NT$1 = 1 點（四捨五入）
2. 折抵規則：100 點 = NT$10（僅限整百點數折抵）
3. 折抵上限：每筆消費最多折抵 20%
4. 計算順序：先套用促銷折扣，再折抵點數（折抵後金額為基礎）
5. 已完成的銷售不追溯補點
6. 既有銷售記錄無需回溯建立會員資料

→ 請確認以上假設後繼續。
```

#### 步驟 1.2：撰寫異動規格文件

```markdown
# 會員點數功能 - 異動規格文件 v1.0

## 1. Objective（目標）
在現有銷售系統中加入會員點數機制，提升顧客回購率。

新增功能邊界（本次異動範圍）：
- IN SCOPE：會員建立、點數累積、點數折抵、點數歷史查詢
- OUT OF SCOPE：
  - 點數到期機制（第二階段）
  - 會員等級制度（第二階段）
  - 行動 App 推播通知（範疇外）

## 2. 相容性要求
- 現有銷售流程（無會員）維持不變（向後相容）
- 現有銷售報表加入「點數折抵金額」欄位（不破壞現有欄位）
- API 版本：新增 /api/v2/sales 支援會員點數，
            /api/sales 保持現有行為（向後相容）

## 3. 資料遷移計劃
- 現有 Sale 記錄：memberId 欄位 nullable（不影響舊資料）
- 無需回溯補點（已確認）
- Migration 可安全在生產環境執行（僅新增欄位 / 表格）

## 4. 測試策略（維護期重點）
- 回歸測試優先：現有 POS 流程不受影響（需 100% 通過）
- 新功能測試：點數計算邏輯的邊界值測試（含促銷 + 點數組合）
- E2E 測試：無會員銷售流程 + 有會員含折抵銷售流程

## 5. 回滾計劃
- 採用 Feature Flag 控制會員功能開關
- 可隨時關閉 Flag 回到無會員模式，無需 Code Rollback
- 資料庫 Migration 可 down（僅新增欄位，rollback 為刪除）
```

---

### Phase 2：PLAN — 異動任務拆解

**對應技能：** `planning-and-task-breakdown`  
**對應指令：** `/plan`

```
異動任務清單（維護期特有注意事項標注）

Task M1.1  資料庫 Migration
  - 新增 Member 表（phone 唯一索引、createdAt）
  - 新增 PointTransaction 表（memberId, saleId, points, type）
  - 修改 Sale 表（新增 nullable 欄位：memberId, pointsEarned, pointsRedeemed）
  ⚠️ 維護期注意：Migration 需在離峰時段執行（凌晨 2-4 點）
  ⚠️ 維護期注意：先在 Staging 環境執行並確認後再上 Production

Task M1.2  Feature Flag 設定（先行完成，作為安全開關）
  - 環境變數：ENABLE_MEMBER_POINTS=false（預設關閉）
  - 所有新功能包在 Flag 判斷內

Task M2.1  會員 Repository（CRUD + 點數查詢）
Task M2.2  點數計算服務（PointsService）
  - 累積計算：Math.floor(amount / 1) = points
  - 折抵計算：Math.floor(pointsToRedeem / 100) * 10
  - 折抵上限驗證：折抵金額 ≤ 銷售金額 × 0.2
  ⚠️ 維護期注意：此模組需 100% 單元測試覆蓋率（業務核心）

Task M2.3  修改 Sale API（v2 端點，保留 v1 向後相容）
  ⚠️ 維護期注意：必須確認 /api/sales（v1）行為完全不變

Task M3.1  POS 介面修改（會員查詢輸入 + 點數顯示）
  ⚠️ 維護期注意：無會員狀態的 POS 流程視覺與操作需維持不變

Task M3.2  會員管理頁面（店長 + 管理員）

Task M4.1  回歸測試套件補強
  - 補充現有 POS 流程的 E2E 測試（若覆蓋率不足）
  
Task M4.2  新功能測試（點數計算邊界值）
Task M4.3  Staging 環境驗收

Task M5.1  分段上線（Feature Flag 控制）
  - 第 1 天：Feature Flag ON，僅限 1 家門市試行 1 週
  - 確認無問題後，逐步開放其他門市
```

---

### Phase 3：BUILD — 維護期特有的實作紀律

**對應技能：** `incremental-implementation` + `test-driven-development`  
**對應指令：** `/build`

#### 向後相容的 API 設計範例

```typescript
// 維護期原則：保留舊端點，新增版本化端點
// app/api/sales/route.ts（v1 - 保持不變）
export async function POST(request: Request) {
  // 原有邏輯，完全不動
  // Feature Flag 確保此端點不受影響
}

// app/api/v2/sales/route.ts（v2 - 新增會員點數支援）
export async function POST(request: Request) {
  if (!process.env.ENABLE_MEMBER_POINTS) {
    // Flag 關閉時，行為等同 v1
    return v1SaleHandler(request);
  }
  
  const body = await request.json();
  const parsed = CreateSaleV2Schema.safeParse(body);
  
  const sale = await db.$transaction(async (tx) => {
    // 1. 建立銷售（同 v1）
    const baseSale = await createBaseSale(tx, parsed.data);
    
    // 2. 若有會員，計算並記錄點數
    if (parsed.data.memberId) {
      const pointsEarned = pointsService.calculate(baseSale.finalAmount);
      const pointsRedeemed = parsed.data.pointsToRedeem ?? 0;
      
      await tx.pointTransaction.createMany({
        data: [
          { memberId: parsed.data.memberId, saleId: baseSale.id,
            points: pointsEarned, type: 'EARN' },
          ...(pointsRedeemed > 0 ? [{
            memberId: parsed.data.memberId, saleId: baseSale.id,
            points: -pointsRedeemed, type: 'REDEEM'
          }] : [])
        ]
      });
    }
    
    return baseSale;
  });
  
  return Response.json(sale, { status: 201 });
}
```

#### 點數計算的嚴謹測試

```typescript
// 維護期原則：業務核心邏輯需要完整的邊界值測試
describe('PointsService', () => {
  describe('累積點數計算', () => {
    it.each([
      [100,   100],   // 整數
      [99.9,  99],    // 小數無條件捨去
      [0,     0],     // 零消費
    ])('消費 %p 元，累積 %p 點', (amount, expected) => {
      expect(pointsService.calculateEarned(amount)).toBe(expected);
    });
  });
  
  describe('折抵金額計算', () => {
    it.each([
      [100,  10],    // 整百點：100點 = NT$10
      [150,  10],    // 非整百，取整：150點 → 100點 → NT$10
      [99,   0],     // 未達 100 點，無法折抵
      [0,    0],     // 零點
    ])('折抵 %p 點，減免 %p 元', (points, expected) => {
      expect(pointsService.calculateRedemption(points)).toBe(expected);
    });
  });
  
  describe('折抵上限（20%）', () => {
    it('折抵超過 20% 應拋出錯誤', () => {
      // 銷售金額 100，20% 上限 = 20 元 = 200 點
      // 嘗試折抵 300 點（= 30 元），超過上限
      expect(() =>
        pointsService.validateRedemption(300, 100)
      ).toThrow('折抵金額超過上限');
    });
    
    it('促銷折扣 + 點數折抵的組合計算', () => {
      // 原價 1000，促銷 9 折 → 900
      // 900 的 20% = 180 元上限 → 最多可折抵 1800 點（取整百 = 1800）
      const afterDiscount = 1000 * 0.9;  // 900
      const maxRedemption = pointsService.getMaxRedemption(afterDiscount);
      expect(maxRedemption).toBe(1800);  // 180 元 = 1800 點
    });
  });
});
```

---

### Phase 4：VERIFY — 維護期回歸驗證

**對應技能：** `debugging-and-error-recovery`  
**對應指令：** `/test`

#### 維護期特有：完整回歸檢查清單

```
回歸測試執行計劃（Feature M 完成後）

【自動化回歸】
npm test -- --testPathPattern="sales|inventory|pos"
→ 確認所有現有測試仍然通過

npx playwright test tests/e2e/sales-flow.spec.ts
→ 確認無會員的 POS 完整流程不受影響

【手動回歸清單（QA 工程師執行）】
□ 無會員銷售：從掃商品到列印收據，流程與上線前一致
□ 促銷折扣銷售：折扣計算結果與上線前一致
□ 庫存扣減：銷售後庫存正確扣減（與會員功能無關）
□ 報表格式：現有報表欄位無新增 / 刪除 / 位移

【新功能驗收】
□ 建立新會員：電話號碼驗證、重複電話拒絕
□ 會員銷售：點數正確累積（NT$299 → 299 點）
□ 點數折抵：NT$299 消費，折抵 100 點 → 實付 NT$289
□ 折抵上限：嘗試折抵超過 20% → 系統拒絕並顯示明確錯誤訊息
□ 促銷 + 折抵組合：促銷後再折抵，計算順序正確

【Feature Flag 測試】
□ ENABLE_MEMBER_POINTS=false：POS 介面無會員輸入欄位
□ ENABLE_MEMBER_POINTS=true：POS 介面顯示會員查詢輸入框
```

---

### Phase 5：REVIEW — 維護期審查重點

**對應技能：** `code-review-and-quality` + `security-and-hardening`  
**對應指令：** `/review`

```
PR 審查報告：feat/member-points（共 1,243 行變更）

【軸 1：正確性】⚠️ Nit → 必須修正
問題：PointsService.calculateRedemption() 使用 Math.round 而非 Math.floor
  // 錯誤
  return Math.round(points / 100) * 10;  // 149點 → 150點 → 15元（錯誤，應為 140→10元）
  // 正確
  return Math.floor(points / 100) * 10;  // 149點 → 100點 → 10元

【軸 2：可讀性】✅
- PointsService 邊界明確，命名清晰
- Feature Flag 邏輯集中管理，易於未來移除

【軸 3：架構】✅
- v1 API 保持不變（向後相容確認）
- PointTransaction 與 Sale 的關係正確建模

【軸 4：安全性】⚠️ Optional（建議修正）
- 點數折抵請求應在 Server 端重新驗證上限，
  不可僅依賴前端傳入的 pointsToRedeem 值
  （前端可被竄改，需 Server 端再次呼叫 validateRedemption）

【軸 5：效能】⚠️ FYI
- Member 表的 phone 欄位已建索引（✅）
- PointTransaction 查詢頻繁，建議對 memberId + createdAt 建複合索引

回歸驗證：✅ 所有現有測試通過（203/203）
結論：修正正確性問題（Math.floor）後可合併。
```

---

### Phase 6：SHIP — 維護期分段上線

**對應技能：** `shipping-and-launch` + `deprecation-and-migration`  
**對應指令：** `/ship`

#### 維護期的上線策略（Feature Flag 控制）

```
Week 1（內部試行）
─────────────────
步驟 1：Staging 環境執行 Migration
  npx prisma migrate deploy --preview-feature
  → 確認 Migration 成功，現有資料完整

步驟 2：Production 環境執行 Migration（離峰時段）
  → 新增欄位均為 nullable，不影響任何現有記錄
  → 現有功能完全正常（Feature Flag 仍為 false）

步驟 3：部署新程式碼（Feature Flag = false）
  → 所有使用者看到的系統與部署前完全相同
  → 確認部署後基本功能正常（15 分鐘觀察期）

Week 2（門市 A 試行）
──────────────────────
步驟 4：僅對門市 A 開啟 Feature Flag
  ENABLE_MEMBER_POINTS=true（僅門市 A 環境）
  → 店長 + 2 名收銀員試用 1 週
  → 蒐集反饋（點數計算是否正確、操作流程是否順暢）

監控指標：
  - 錯誤率（Sentry）：維持 < 0.1%
  - 銷售 API 回應時間：P95 < 300ms（加入點數計算後）
  - 點數計算錯誤回報：0 件

Week 3（全門市上線）
──────────────────────
確認 Week 2 無問題後：
  ENABLE_MEMBER_POINTS=true（全部門市）

【回滾觸發條件】
- 任何銷售資料計算錯誤 → 立即關閉 Flag
- 錯誤率超過 0.5% → 立即關閉 Flag
- 任何點數資料不一致 → 立即關閉 Flag + 通知所有門市

【Feature Flag 移除計劃（第二階段，穩定 1 個月後）】
- 移除 Flag 判斷程式碼（使用 deprecation-and-migration 技能）
- 清理 v1 API（提供 3 個月遷移期，若有外部整合）
```

#### 上線結果

```
會員點數功能 v1.0 完整上線

開發時程：3 週（含 1 週 Staging 驗證）
上線策略：Feature Flag 分段推出（門市 A 先行 → 全門市）
回歸影響：現有銷售功能零中斷

營運指標（上線 2 週後）：
  ✅ 累積會員 387 人
  ✅ 點數折抵使用率 23%（387 人中有 89 人使用折抵）
  ✅ 點數計算錯誤回報：0 件
  ✅ 現有銷售流程：完全無影響
```

---
---

## 附錄：技能載入策略

### 最小配置（建議新團隊從這裡開始）

將以下三個技能加入 `.github/copilot-instructions.md` 或 `CLAUDE.md`：

```markdown
[skills/spec-driven-development/SKILL.md 的內容]
[skills/test-driven-development/SKILL.md 的內容]
[skills/code-review-and-quality/SKILL.md 的內容]
```

### 情境式載入（進階）

| 工作情境 | 載入技能 |
|---------|---------|
| 開始新功能 | `spec-driven-development` → `planning-and-task-breakdown` |
| 實作中 | `incremental-implementation` + `test-driven-development` |
| 維護期異動 | 以上全部 + `deprecation-and-migration` |
| 偵錯中 | `debugging-and-error-recovery` |
| 合併前 | `code-review-and-quality` + `security-and-hardening` |
| 準備部署 | `shipping-and-launch` |
| 程式碼太複雜 | `code-simplification` |

### 技能的關鍵作用：防止 AI 代理的常見失誤

| 常見失誤 | 對應技能的防護機制 |
|---------|-----------------|
| 先寫程式再想規格 | `spec-driven-development`：寫程式前必須通過人工確認規格閘門 |
| 一次寫完所有程式 | `incremental-implementation`：強制薄片垂直切片 |
| 先寫程式再補測試 | `test-driven-development`：強制 Red-Green-Refactor 順序 |
| 遇到錯誤繼續開發 | `debugging-and-error-recovery`：Stop-the-Line 規則 |
| 跳過安全性考量 | `security-and-hardening`：OWASP Top 10 檢查清單 |
| 合併未經審查的程式碼 | `code-review-and-quality`：五軸審查閘門 |
| 直接全量上線 | `shipping-and-launch`：分段推出 + 回滾程序 |
| 維護期改壞現有功能 | `deprecation-and-migration`：向後相容 + Feature Flag |

---

*本手冊基於 `agent-skills` 專案，詳細技能定義請參閱 `skills/` 目錄中各技能的 `SKILL.md`。*
