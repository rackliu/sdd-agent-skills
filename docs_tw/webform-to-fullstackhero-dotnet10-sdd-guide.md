# WebForm 到 FullStackHero .NET 10 重建開發指南（SDD Agent Skills 版）

## 0. 目標與適用範圍

本文件提供一套可直接執行的 Step By Step 流程，協助團隊將早期 .NET 1.1 WebForm 系統，使用 FullStackHero .NET 10 Starter Kit 重新開發，並完整套用本倉庫的 SDD Agent Skills。

適用情境：
- 舊系統為 ASP.NET WebForm（含 code-behind、ViewState、Server Controls、舊式 DataSet/SqlDataAdapter）
- 團隊要重建為現代化 API 為核心架構（.NET 10）
- 需要在需求不完整、文件缺漏、歷史技術債高的情況下，仍能控制範圍、品質與交付節奏

非目標：
- 不做「原樣升級」到新版 WebForm
- 不鼓勵一次性 Big Bang 全量切換

---

## 1. 先備資訊與環境設定

### 1.1 團隊角色建議

- 產品/業務代表：確認流程、規則、驗收條件
- 遷移負責工程師：主導規格、拆解、實作節奏
- 測試負責工程師：建立回歸驗證策略
- 平台/DevOps：CI/CD、環境、監控與上線策略

### 1.2 工具與基礎環境

- .NET 10 SDK
- Docker（供 Aspire 啟動 Postgres/Redis）
- FullStackHero CLI（建議）
- Git 與分支策略
- 既有 WebForm 原始碼與資料庫存取權限

### 1.3 FullStackHero 快速初始化（建議）

~~~bash
dotnet tool install -g FullStackHero.CLI
fsh doctor
fsh new LegacyModernization --db sqlserver
cd LegacyModernization
dotnet run --project src/Playground/LegacyModernization.AppHost
~~~

備選方式：

~~~bash
dotnet new install FullStackHero.NET.StarterKit
dotnet new fsh -n LegacyModernization
cd LegacyModernization
dotnet run --project src/Playground/LegacyModernization.AppHost
~~~

說明：
- 若舊系統資料庫是 SQL Server，建立專案時優先選擇 SQL Server 提供者，降低遷移摩擦
- 先以 Playground/AppHost 啟動，確保底座可執行，再開始業務模組遷移

---

## 2. SDD 執行總流程（建議固定節奏）

每個功能或子系統，固定走以下生命週期：

1. DEFINE：spec-driven-development
2. PLAN：planning-and-task-breakdown
3. BUILD：incremental-implementation + test-driven-development
4. VERIFY：debugging-and-error-recovery
5. REVIEW：code-review-and-quality（必要時加 security-and-hardening、performance-optimization）
6. SHIP：documentation-and-adrs + shipping-and-launch + deprecation-and-migration

關鍵原則：
- 任何非 trivial 工作，先規格、再實作
- 每次只遷移一條可驗證垂直切片
- 以測試與對照驗證保護舊行為
- 嚴禁在不確定需求時硬寫

---

## 3. Step By Step 詳細步驟

## Step 1：舊系統盤點與風險分級（Discovery）

對應技能：
- spec-driven-development（前置澄清）
- context-engineering（整理上下文）

要做的事：
1. 盤點頁面與使用流程：登入、查詢、交易、報表、批次作業
2. 盤點資料表、Stored Procedure、排程、外部整合
3. 標記高風險功能：金流、權限、結帳、庫存、結算
4. 識別 WebForm 特有耦合：ViewState、PostBack、Server Events、Session 相依

交付物：
- Legacy Inventory 文件
- 功能風險矩陣（High/Medium/Low）
- 關鍵流程清單（Top 10）

驗收標準：
- 至少 80% 主流程被識別
- 每個高風險功能有對應資料來源與負責人

與 AI 溝通提示詞對話參考：

你可以這樣問 AI（盤點啟動）：
我現在要做 Step 1 舊系統盤點，請依照 spec-driven-development 協助我先輸出盤點框架。
背景：系統是 .NET 1.1 WebForm，請你先列出你對此系統的關鍵假設，並產出以下清單模板：
1) 頁面與流程盤點表
2) 資料表與 Stored Procedure 盤點表
3) 外部整合盤點表
4) 風險分級矩陣（High/Medium/Low）
最後請列出你還需要我補充的資料。

你可以這樣追問 AI（缺漏補齊）：
以下是我已盤點內容，請幫我檢查盲點，特別是 WebForm 常見耦合：ViewState、PostBack、Session、Server Events。
請輸出：
1) 缺漏項目清單
2) 每個缺漏對遷移風險的影響
3) 建議優先補查順序

你預期 AI 回覆格式：
- Assumptions
- Inventory Tables
- Risk Matrix
- Gaps and Next Actions

---

## Step 2：定義目標架構與遷移策略（Spec）

對應技能：
- spec-driven-development
- api-and-interface-design

要做的事：
1. 定義目標架構：FullStackHero 模組化 + Vertical Slice + Minimal API
2. 明確切割界線：哪些需求保留、哪些淘汰、哪些重設流程
3. 規範 API 合約：Request/Response、錯誤碼、版本策略
4. 定義不可破壞規則（Always / Ask First / Never）

建議目標對映：
- WebForm Page + code-behind 邏輯 -> Feature Slice（Command/Query + Handler + Endpoint）
- Repeater/GridView 的資料操作 -> Query Endpoint + 分頁排序過濾
- 頁面事件（Button_Click）-> Command Endpoint
- Session/Cache 依賴 -> Token + 伺服器端快取策略

交付物：
- SPEC 文件（可放於 docs 或 tasks）
- API 契約草案
- 邊界規則

驗收標準：
- 成功條件可測試（非描述性）
- 至少 1 個完整流程有端到端規格

與 AI 溝通提示詞對話參考：

你可以這樣問 AI（規格產出）：
請依照 spec-driven-development，為 WebForm -> FullStackHero .NET 10 遷移產出 SPEC 初稿。
請包含：Objective、Commands、Project Structure、Code Style、Testing Strategy、Boundaries、Success Criteria、Open Questions。
限制：
1) 成功條件必須可量測
2) 每條邊界規則都要有理由
3) 所有不確定項目放在 Open Questions

你可以這樣追問 AI（架構對映）：
請把以下 WebForm 元件對映到 FullStackHero 架構：Page、code-behind、GridView、Button_Click、Session。
請輸出對映表，欄位包含：舊元件、目標模組、目標 Endpoint、測試方式、風險。

你預期 AI 回覆格式：
- SPEC Draft
- Mapping Table
- Open Questions
- Approval Checklist

---

## Step 3：任務拆解與里程碑排程（Plan）

對應技能：
- planning-and-task-breakdown

要做的事：
1. 依賴導向拆解任務（資料層 -> 應用層 -> API 層 -> 前端/整合）
2. 每個任務限制在 1 到 5 個檔案變更為主
3. 每個任務都要有 Acceptance 與 Verify 指令
4. 建立每 2 到 3 個任務一個 Checkpoint

建議切片順序：
1. 身分驗證與授權
2. 主查詢流程
3. 主交易流程
4. 報表/匯出
5. 後台維運功能

交付物：
- Plan 文件
- Todo 清單（含依賴、估時、驗證方式）

驗收標準：
- 不存在 XL 任務（8+ 檔案且無法一次驗證）
- 所有任務可獨立測試

與 AI 溝通提示詞對話參考：

你可以這樣問 AI（任務拆解）：
請依照 planning-and-task-breakdown，將目前 SPEC 拆成可執行任務。
要求：
1) 每個任務包含 Acceptance、Verify、Dependencies、Files likely touched
2) 每個任務盡量 1-5 個檔案
3) 每 2-3 個任務建立 Checkpoint
4) 先做高風險高價值流程

你可以這樣追問 AI（可平行處理分析）：
請標記任務中哪些可以平行處理，哪些必須序列執行，並說明依賴原因。
另外請給出第一個 Sprint 的建議任務順序。

你預期 AI 回覆格式：
- Ordered Task List
- Dependency Graph (text)
- Parallelization Notes
- Sprint 1 Proposal

---

## Step 4：建立基線測試與行為對照（Test Baseline）

對應技能：
- test-driven-development
- debugging-and-error-recovery

要做的事：
1. 先為舊系統關鍵流程建立 Golden Master 對照資料
2. 對新系統先寫會失敗的測試（Red）
3. 實作最小功能讓測試通過（Green）
4. 重構（Refactor）且確保測試持續通過

建議測試分層：
- 單元測試：商業規則、驗證邏輯
- 整合測試：資料存取、交易邊界、外部服務替身
- 合約測試：API schema 與錯誤回應一致性
- 端到端測試：關鍵流程（最少 Top 5）

交付物：
- 測試基線報告
- Golden Master 對照結果

驗收標準：
- 關鍵流程皆可被測試腳本重現
- 每次切片完成都可自動驗證

與 AI 溝通提示詞對話參考：

你可以這樣問 AI（測試基線建立）：
請依照 test-driven-development，幫我為以下關鍵流程建立測試基線策略。
流程：登入、查詢訂單、建立訂單、審核、匯出報表。
請輸出：
1) Golden Master 對照資料建議
2) 每個流程的 Red 測試案例
3) 測試層級分配（單元/整合/合約/E2E）

你可以這樣追問 AI（失敗測試轉修復）：
以下是目前失敗測試輸出，請依照 debugging-and-error-recovery 的流程，告訴我重現步驟、根因假設、最小修復方案與防回歸測試。

你預期 AI 回覆格式：
- Baseline Test Plan
- Red Cases
- Fix Hypotheses
- Regression Guardrails

---

## Step 5：逐片遷移（Incremental Build）

對應技能：
- incremental-implementation
- test-driven-development
- source-driven-development

每個切片固定步驟：
1. 選 1 條使用者流程（例如：建立訂單）
2. 補齊該流程的失敗測試
3. 實作對應 Command/Query + Handler + Endpoint
4. 完成資料映射與驗證規則
5. 執行測試、修正、重構
6. 提交小而可回滾的變更

建議控制：
- 單次 PR 盡量小，便於審查
- 新舊系統並行期間使用 Feature Flag
- 禁止未驗證即合併

交付物：
- 每個切片的 PR
- 測試與驗證紀錄

驗收標準：
- 切片可以獨立發布或回退
- 舊行為與新行為差異有明確說明

與 AI 溝通提示詞對話參考：

你可以這樣問 AI（單一切片實作）：
請依照 incremental-implementation + test-driven-development，實作 Task [編號]。
先做：
1) 失敗測試
2) 最小可行實作
3) 重構
完成後請回報：變更摘要、測試結果、未解風險、是否可回退。

你可以這樣追問 AI（PR 整理）：
請幫我把這次切片整理成可審查 PR 說明。
內容要包含：目的、範圍、主要變更點、相容性影響、驗證證據、回退方式。

你預期 AI 回覆格式：
- Implementation Steps
- Changed Files Summary
- Test Evidence
- Rollback Notes

---

## Step 6：資料遷移與雙軌運行（Migration）

對應技能：
- deprecation-and-migration
- ci-cd-and-automation

要做的事：
1. 設計資料映射表（舊欄位 -> 新模型）
2. 定義資料清洗規則（空值、重複、非法值）
3. 建立遷移腳本與可重放流程
4. 先在 Stage 執行彩排
5. 實施雙軌期（新舊讀寫策略需明確）

雙軌策略範例：
- Read from old, write to new（過渡期）
- 對帳作業每日執行
- 差異超過門檻即停止切換

交付物：
- Data Migration Runbook
- 對帳報表與差異處理手冊

驗收標準：
- 遷移流程可重複執行
- 對帳誤差低於門檻且有修復流程

與 AI 溝通提示詞對話參考：

你可以這樣問 AI（遷移計畫）：
請依照 deprecation-and-migration，為我建立資料遷移 Runbook。
背景：舊資料庫到 FullStackHero 新模型。
請輸出：
1) 欄位映射表
2) 清洗規則
3) 可重放遷移步驟
4) 對帳腳本策略
5) 失敗回復方案

你可以這樣追問 AI（雙軌運行）：
請設計 2 週雙軌運行方案，包含每日對帳、差異告警門檻、停止切換條件、Go/No-Go 判準。

你預期 AI 回覆格式：
- Migration Runbook
- Reconciliation Strategy
- Rollback and Stop Conditions
- Go/No-Go Criteria

---

## Step 7：品質審查與安全強化（Review）

對應技能：
- code-review-and-quality
- security-and-hardening
- performance-optimization

要做的事：
1. 五軸審查：正確性、可讀性、架構、安全、效能
2. 安全檢查：認證授權、輸入驗證、敏感資料保護
3. 效能檢查：慢查詢、快取命中率、API 延遲

交付物：
- 審查報告（含 Severity）
- 修正清單

驗收標準：
- 不存在阻斷級風險
- 效能指標達成目標（例如 P95 延遲）

與 AI 溝通提示詞對話參考：

你可以這樣問 AI（品質審查）：
請依照 code-review-and-quality 對目前變更做審查。
輸出順序必須是：
1) Findings（依嚴重度排序）
2) Open Questions
3) 建議修正
最後才給摘要。

你可以這樣追問 AI（安全與效能）：
請追加 security-and-hardening 與 performance-optimization 觀點，列出：
1) 可能的 OWASP 風險
2) 效能瓶頸與量測方法
3) 修正優先順序

你預期 AI 回覆格式：
- Findings by Severity
- Security Risks
- Performance Risks
- Recommended Fix Order

---

## Step 8：上線切換與舊系統退場（Ship）

對應技能：
- shipping-and-launch
- documentation-and-adrs
- deprecation-and-migration

要做的事：
1. 先小流量釋出（Canary）
2. 監控關鍵指標（錯誤率、延遲、交易成功率）
3. 逐步放量
4. 保留明確回滾方案
5. 完成舊系統退場計畫

交付物：
- 上線檢查清單
- 回滾手冊
- ADR 與最終遷移報告

驗收標準：
- 上線期間無重大事故
- 退場後無未受控依賴

與 AI 溝通提示詞對話參考：

你可以這樣問 AI（上線前檢查）：
請依照 shipping-and-launch，幫我建立本次遷移上線 Checklist。
請包含：部署前檢查、Canary 計畫、監控指標、告警門檻、回滾條件、溝通窗口。

你可以這樣追問 AI（退場計畫）：
請依照 deprecation-and-migration，幫我產出舊系統退場計畫。
內容包含：退場前條件、並行期間、停用順序、資料封存、法遵稽核需求、最終下線步驟。

你預期 AI 回覆格式：
- Go-Live Checklist
- Canary Plan
- Rollback Playbook
- Decommission Plan

---

## 4. FullStackHero 對應實作重點（遷移時常見）

### 4.1 模組切分建議

- Identity：登入、權限、使用者管理
- Multitenancy：若舊系統有多客戶資料隔離需求
- Auditing：關鍵操作與異常追蹤
- Webhooks：對外事件整合

### 4.2 應用程式啟動與驗證

~~~bash
dotnet restore src/FSH.Starter.slnx
dotnet test src/FSH.Starter.slnx
dotnet run --project src/Playground/FSH.Starter.AppHost
~~~

### 4.3 API only 執行（必要時）

~~~bash
dotnet run --project src/Playground/FSH.Starter.Api
~~~

---

## 5. 建議文件結構與產出管理

建議在專案根目錄維持以下文件：

- docs/migration/SPEC.md
- docs/migration/PLAN.md
- docs/migration/TODO.md
- docs/migration/ADR-*.md
- docs/migration/RUNBOOK.md
- docs/migration/GO-LIVE-CHECKLIST.md

建議流程：
- 任何需求與設計變更，先更新 SPEC/PLAN，再修改程式碼
- 每個里程碑更新風險與驗證狀態

---

## 6. 可直接貼用的提示詞範本（給 AI 代理）

## 6.1 規格階段提示詞

請依照 spec-driven-development 幫我產出遷移規格。
背景：我們要把 .NET 1.1 WebForm 系統遷移到 FullStackHero .NET 10 Starter Kit。
請輸出：Objective、Commands、Project Structure、Code Style、Testing Strategy、Boundaries、Success Criteria、Open Questions。
先列出你目前假設，再等我確認。

## 6.2 拆解階段提示詞

請依照 planning-and-task-breakdown，把 SPEC 拆成可執行任務。
要求：每個任務有 Acceptance、Verify、Dependencies、Files likely touched。
每個任務盡量維持小範圍，並設置每 2-3 個任務一個 Checkpoint。

## 6.3 實作階段提示詞

請依照 incremental-implementation + test-driven-development，先做 Task X。
先寫失敗測試，再做最小實作，最後重構。
完成後回報：變更檔案、測試結果、尚未解決風險。

## 6.4 審查階段提示詞

請依照 code-review-and-quality 進行審查。
先列出 Findings，按嚴重度排序，附檔案與位置。
再列出 Open Questions，最後才給變更摘要。

---

## 7. 里程碑建議（12 週範例）

- 第 1-2 週：盤點與規格（Step 1-2）
- 第 3-4 週：計畫拆解與測試基線（Step 3-4）
- 第 5-9 週：核心流程逐片遷移（Step 5）
- 第 10 週：資料遷移彩排（Step 6）
- 第 11 週：品質、安全、效能關卡（Step 7）
- 第 12 週：分段上線與退場啟動（Step 8）

---

## 8. 風險與對策速查表

- 需求不完整 -> 先完成 SPEC 的 Open Questions 再進 BUILD
- 舊程式邏輯不可見 -> 建 Golden Master 與行為對照測試
- 資料品質差 -> 先做 Profiling 與清洗規則
- 一次改太多 -> 強制垂直切片與小 PR
- 上線風險高 -> Feature Flag + Canary + 可演練回滾

---

## 9. 完成定義（Definition of Done）

同時滿足以下條件才算完成：

1. 所有高優先流程已在新系統可執行
2. 自動化測試、整合測試、關鍵 E2E 全數通過
3. 資料遷移對帳達標
4. 已完成安全與效能審查
5. 已有可驗證回滾方案
6. 已完成 ADR 與運維文件
7. 舊系統退場計畫已核准並排程

---

## 10. 實務建議

- 先搬商業核心流程，不先搬低價值頁面
- 對「看起來只是畫面」的 WebForm 頁面仍要追資料與規則來源
- 任何隱含規則都要轉成可測試條件
- 每週固定一次規格回顧，避免邊做邊失焦
- 保持文件活著：Spec/Plan 不是一次性產物

本文件可作為團隊標準作業程序（SOP），建議直接納入專案內部開發手冊。