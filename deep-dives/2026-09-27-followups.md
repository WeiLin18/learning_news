# 深入追問筆記 2026-09-27

這份文件整理使用者直接提供的 Cloudflare OS 與 Software Factories 文章，補上比新聞摘要更完整的機制解釋、具體例子與採用判斷；兩篇原文都未曾收錄於既有 `digests/`。

## 目錄

1. [Cloudflare OS：真正的新東西不是聊天介面，而是可追蹤資料流向的權限模型](#1-cloudflare-os真正的新東西不是聊天介面而是可追蹤資料流向的權限模型)（追問對象：使用者直接提供）
2. [Software Factories：產生程式碼不再是瓶頸，驗證才是](#2-software-factories產生程式碼不再是瓶頸驗證才是)（追問對象：使用者直接提供）

---

## 1. Cloudflare OS：真正的新東西不是聊天介面，而是可追蹤資料流向的權限模型

### 一句話定位

Cloudflare OS 想做的不是傳統作業系統，也不只是「公司版 ChatGPT」；它把 agent、公司知識、內部系統與臨時小工具放進同一個工作空間，並在底層統一處理執行環境、權限、稽核與成本。

可以把它想成一棟辦公大樓：agent 是新進助理，MCP 是「助理會使用哪些辦公設備」，Gatekeeper 則是每個房間的門禁與警衛。只知道助理會用影印機還不夠；如果他剛看過薪資表，影印出來的報表也不能隨手放到公開會議室。Cloudflare OS 最有意思的主張正是：**授權不能只管 agent 能呼叫什麼，還要跟著它看過的資料與產物一起移動。**

### 三層架構怎麼拼起來

| 層次 | 負責什麼 | 白話例子 |
|---|---|---|
| Workspace | 保存對話、檔案、公司 skills、執行狀態 | 每位員工有一張不會下班就清空的工作桌 |
| Gadget / Blueprint | agent 產生的完整小型 app；Blueprint 只分享程式碼，不帶資料與憑證 | 把「客訴儀表板」分享成模板，別人複製後接自己的資料 |
| Gatekeeper | 代管 OAuth、縮窄資源與操作、記錄讀寫、要求人工核准 | 只准讀某個 repo 的 issues，不准讀 source code 或直接 merge |

每個 Gadget 都有瀏覽器端 UI、server code、API 和獨立 SQLite 狀態。Server 以 Dynamic Worker 啟動，狀態放在 Durable Object Facet；client 與 server 透過 Cap'n Web RPC 溝通。這讓同一個方法既能由按鈕呼叫，也能由 agent 呼叫：

```ts
// 使用者按「只看已完成」或 agent 自動整理週報，都走同一個受控介面
const issues = await app.listIssues({ status: "done" });

// 關鍵不是語法，而是 agent 拿不到 GitHub token；
// 它只拿到被 Gatekeeper 限縮後的能力。
```

### 權限為什麼比一般 MCP 多一層

一般 MCP server 可以隱藏真正的 API key，卻未必知道 agent 從工具回傳值裡「看過哪一筆資源」。Cloudflare OS 讓 agent 預設零權限，使用者再把具體資源介紹給它；產生的程式只取得 typed binding，不取得憑證：

```ts
// 只代表「依目前 policy 讀 PROJECT 的 issues」，不是整個 GitHub 帳號
const issues = await env.PROJECT.listIssues({
  teamId: "ENG",
  state: "open",
});
```

假設 agent 讀過未公開的人事表，再做成「各部門平均薪資」儀表板。傳統 RBAC 常只檢查「分享者能否分享這個 dashboard」；Cloudflare OS 會把來源觀察紀錄留在 workspace 與產物上，接收者開啟時還要通過原始資源的授權。這接近資料流控制（information-flow control），但目前公開說明仍以產品行為為主，尚不足以把它視為經過形式驗證、能涵蓋所有推論洩漏的安全保證。

### 它真正想改變的工作方式

原文把已知步驟交給 code，只在需要判斷的節點呼叫模型。以「每週客服摘要」為例：查詢、去重、分組與排程都應是 deterministic workflow；只有判斷客訴語氣與歸納共通原因才需要 LLM。這比每週重跑一個長 prompt 更便宜、可測，也較不會因模型漂移而改變結果。

另一個重要設計是「分享 app」與「分享 Blueprint」分開。前者多人操作同一份狀態；後者只複製程式碼，刻意不帶 SQLite 資料、對話、憑證或已連接資源。它把 SaaS 的中央單一實例，改成每個人都有可被 AI 修改的私人副本；好處是客製快速，代價則是版本分岔、品質控管與維護責任可能從平台團隊擴散到每位使用者。

### 現在值不值得導入

- **適合試驗**：公司已在用 Workers、想讓非工程人員做內部小工具，而且最在意最小權限、稽核與資料外流控制。
- **先做小型 PoC**：選一個唯讀、可驗證的流程，例如從單一 GitHub repo 讀 issues 做週報；先量測錯誤率、token 成本、人工核准負擔與權限紀錄是否真的看得懂。
- **不適合直接承擔核心流程**：README 明確把 2026 年 8 月版本標為 early access、仍有 rough edges；自行部署到 `workerd` 的正式文件也尚未完成，外部服務的 OAuth 與各 Gatekeeper 仍需要逐一設定。
- **別忽略平台綁定**：程式碼是 Apache-2.0、`workerd` 也開源，但架構深度依賴 Workers、Durable Objects、Dynamic Workers、Facets、Access 與 AI Gateway。法律上的 open source 不等於搬到其他雲就很輕鬆。

最精簡的結論：Cloudflare OS 最值得學的不是 UI，而是「agent 預設零權限、憑證永不交給生成程式、政策跟著已觀察資料走」這三件事。即使不採用整套產品，設計公司內部 agent 平台時，也應把這三項當成架構檢查表。

### 延伸閱讀

- [Cloudflare 原文](https://blog.cloudflare.com/cloudflare-os/)
- [Cloudflare OS 開源 repo](https://github.com/cloudflare/cloudflare-os)
- [Cloudflare OS starter repo](https://github.com/cloudflare/cloudflare-os-starter)
- [Dynamic Workers 文件](https://developers.cloudflare.com/dynamic-workers/)
- [MCP Server Portals 文件](https://developers.cloudflare.com/cloudflare-one/access-controls/ai-controls/mcp-portals/)

---

## 2. Software Factories：產生程式碼不再是瓶頸，驗證才是

### 一句話定位

Software factory 不是一個超強 agent，而是很多受約束的 agent loop 同時從工作佇列取件，經過自動檢查與 review gate，再部署到 production 的整套生產線。Addy 的重點不是「AI 可以大量寫 code」，而是：**生成已經便宜到近乎無限，稀缺資源變成人類能可靠驗證多少。**

生活化類比是餐廳廚房。買十台自動炒菜機，出菜速度會暴增；但如果只有一位主廚能試味道、檢查過敏原，桌上只會堆滿更多等驗收的菜。拿掉主廚確實能讓出菜數字變漂亮，卻會把錯誤直接送到客人桌上。

### Loop、Harness、Factory 的差別

| 層級 | 做什麼 | 最小例子 |
|---|---|---|
| Loop | 收集 context → 行動 → 檢查 → 重試 | 修一個 lint error，直到測試通過 |
| Harness | 限制 loop 的工具、sandbox、記憶與完成條件 | 只能改指定目錄，最多重試三次，必須跑測試 |
| Factory | 平行執行多個 harnessed loops，統一經過 queue、review、deploy、monitoring | 自動處理多張 issue，但所有 PR 都進同一套品質閘門 |

最小流程不應是「agent 自己想辦法直到完成」，而應把控制流畫成 graph：

```text
重現 bug ─失敗→ 要求更多資訊
   │成功
   ▼
診斷原因 → 實作修正 → 測試
                       ├─失敗→ 回到實作
                       └─成功→ 人工 review → merge
```

Agent 仍能在「診斷」與「實作」節點內發揮，但不能跳過測試，也不能自行把「測不出來」解釋成完成。這正是原文說的 back pressure：自治範圍只能擴張到你能便宜、頻繁且可靠驗證的地方。

### Light factory 與 Dark factory

Dark factory 指沒有人閱讀程式碼，生成、驗證與上線都交給機器；Light factory 則把人類判斷留在錯誤代價高的位置。真正差異不只是 PR 最後有沒有 reviewer，而是人在 agent 動工前是否先審設計與架構。

例如，與其最後面對 2,000 行 generated diff，先花一小時審一份 200 行計畫：資料模型是否合理、API contract 能不能變、migration 如何回滾。這不是保守地拖慢 AI，而是用較便宜的上游決策，避免昂貴的下游返工。

Dark factory 最大的隱性成本是 comprehension debt：code、tests 與 PR 數量持續增加，但沒有任何人能解釋系統為何這樣設計。測試全綠也不能證明架構長期正確，因為測試只能檢查團隊已經想到的行為；agent 若同時改實作與測試，兩者甚至可能一致地錯。

### 哪些 loop 可以關燈？

| 可以高度自動化 | 應保持人工判斷 |
|---|---|
| formatter、明確 lint rule、單一 optional prop 清理 | auth、billing、權限與公開 API contract |
| 小範圍、可回滾、diff 很短 | blast radius 大、難以回滾 |
| 有立即、穩定、難以欺騙的 pass/fail oracle | 正確性需要產品、架構或領域判斷 |
| property test 能完整表達 invariant | 成敗要幾個月後才看得出來的設計決策 |

一個可安全夜跑的例子：每天只修一種 ESLint violation，每次只開一個小 PR，不自動 merge。反例是「把登入模組升級並自行部署」；即使 unit tests 通過，session migration、撤銷機制與跨服務相容性仍需要真正理解系統的人決策。

### 可以直接照做的最小工廠

1. **只選一種低風險工作**：例如修正 `no-unused-vars`，不要從「自動處理所有 issue」開始。
2. **把完成條件寫成機器可判定的 oracle**：限定路徑、lint 與 tests 全綠、diff 不超過 100 行、不得新增 dependency。
3. **隔離執行環境**：每個任務使用獨立 branch/worktree，限制網路、secrets 與可寫目錄。
4. **maker 與 checker 分離**：實作者不能是唯一 reviewer；checker 依固定 rubric 檢查範圍、行為與風險。
5. **先只開 PR，不自動 merge**：記錄誤報率、退回原因與人類 review 時間。只有當 oracle 長期穩定，才逐步增加自治。

這和 `deep-dives/2026-08-02-followups.md` 第 4 則 Addy Osmani LLM coding workflow 是同一條路線：先寫規格、拆小步、測試、review，再前進。Software factory 只是把那個單人流程放大成多條平行生產線；如果沒有同時放大驗證能力，增加 agent 只會增加待審 PR 與理解債。

最精簡的結論：不要用「agent 產出多少」衡量工廠，而要看「多少變更能以低成本被可靠證明」。人沒有離開開發流程，只是從流水線上的打字員，移到上游設計控制流與下游守住 review gate。

### 延伸閱讀

- [Software Factories, Light and Dark](https://addyosmani.com/blog/software-factories/)
- [Loop Engineering](https://addyosmani.com/blog/loop-engineering/)
- [Comprehension Debt](https://addyosmani.com/blog/comprehension-debt/)
- [HumanLayer：12-Factor Agents](https://github.com/humanlayer/12-factor-agents)
