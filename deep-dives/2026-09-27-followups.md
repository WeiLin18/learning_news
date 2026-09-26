# 深入追問筆記 2026-09-27

這份文件整理使用者直接提供的 Cloudflare OS 文章，補上比新聞摘要更完整的機制解釋、具體例子與採用判斷；原文未曾收錄於既有 `digests/`。

## 目錄

1. [Cloudflare OS：真正的新東西不是聊天介面，而是可追蹤資料流向的權限模型](#1-cloudflare-os真正的新東西不是聊天介面而是可追蹤資料流向的權限模型)（追問對象：使用者直接提供）

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
