# 深入追問筆記 2026-10-14

這份文件整理使用者針對 `digests/2026-10-12.md` 第 8 則的追問，補上比日報摘要更完整的作者背景介紹與文章導讀。跟 `digests/` 不同：`digests/` 是每次固定 8 則的精選摘要，這份文件是針對其中特定幾則的深入展開，只在使用者主動追問時才會產生。

## 目錄

1. [The Code Nobody Reads — 完整導讀 + 作者介紹](#1-the-code-nobody-reads--完整導讀--作者介紹)（追問對象：`digests/2026-10-12.md` 第 8 則）

---

## 1. The Code Nobody Reads — 完整導讀 + 作者介紹

> ⚠️ 未能直接開啟原文（Substack 被此環境的 egress policy 封鎖），以下內容依搜尋引擎摘要、多篇轉載摘要及 Addy Osmani 相關文章的交叉引述推得，細節請以原文為準。文章連結：[addyo.substack.com/p/the-code-nobody-reads](https://addyo.substack.com/p/the-code-nobody-reads)

---

### Addy Osmani 是誰

**一句定位**：前端效能與工程領導力領域的長期意見領袖，在 Google Chrome 做了 13+ 年，現在是 Chrome Developer Experience 的負責人。

他有三個最廣為人知的身份：

**1. 工具作者**：[Lighthouse](https://developer.chrome.com/docs/lighthouse/overview/)（網頁效能測量工具，Chrome DevTools 和 PageSpeed Insights 底層都在跑它）和 [Workbox](https://developers.google.com/web/tools/workbox)（PWA 離線快取工具）都出自他的團隊。你用 DevTools Lighthouse 分頁看過你網站跑分，那個計算方式有很大一部分是他定的。

**2. 技術書作者**：他寫了好幾本工程師常當參考書的書：

| 書名 | 年份 | 主題 |
|---|---|---|
| *Learning JavaScript Design Patterns* | 2012 | 把 GoF 設計模式翻譯到 JavaScript，[有免費網頁版](https://www.patterns.dev/) |
| *Image Optimization* | 2021 | 網頁圖片最佳化（Smashing Magazine 出版） |
| *Leading Effective Engineering Teams* | 2024 | 工程師轉型工程主管的指南（O'Reilly） |
| *Beyond Vibe Coding: From Coder to AI-Era Developer* | 2026 | AI 時代工程師角色的轉型——他現在電子報在寫的主題的完整版 |

**3. AI 時代的技術評論者**：2025 年起在 Substack 發 *Elevate* 電子報，主題是「AI agent 大量介入之後，工程師的角色如何調整」。*The Code Nobody Reads* 就是這個系列的一部分。

---

### 文章的脈絡：他今年在說什麼

讀這篇之前，先看他今年寫了什麼，有助於理解這篇在整個論述裡的位置：

- **六月** — *Agentic Code Review*：AI agent 大量介入 code review 時，「信任」怎麼建立。
- **九月中** — *Brownfield Agentic Engineering*：在既有系統上用 AI agent 時，autonomy 應該跟著 blast radius（錯誤影響範圍）走，不是跟著 model 的信心走。
- **九月底（本篇）** — *The Code Nobody Reads*：逐行 review 正在式微，但「對 ship 決定負責」這件事不能消失。

三篇合起來是同一個主張：**AI coding 真正難的地方已經從「怎麼寫出程式碼」移到「怎麼決定要不要信任它」。**

---

### 核心主張：「閱讀程式碼」≠「為程式碼負責」

這是整篇文章最重要的切割。他說他現在給的建議跟以前不一樣了——以前他推「review 每一行」。現在他認為：**一個人仍然要為「要不要 ship」這個決定負責，但不一定要逐行讀才能負起這個責任。**

用兩個比喻對比：

- **舊思維**：負責任 = 親自讀每一行 = 你看過才算審過。像餐廳主廚每道菜都要親口試味。
- **新思維**：負責任 = 你有可信賴的品管機制 + 你的名字在那個 ship 決定上。像食品工廠廠長——他不可能試每一罐，但他設計了 HACCP 流程，出了問題他要去解釋。

**為什麼 AI 程式碼需要這個新思維？**

舊有的程式碼產生器（code template、code gen tool）可以被「認證」，因為同樣的輸入永遠給同樣的輸出，驗一次就放心。AI 不一樣——同樣的 prompt 給不同的答案，沒辦法驗一次就永久信任。所以信任必須來自「繞著程式碼蓋的那些檢查機制」，不是來自有人讀過它。

---

### 他引用的數據

（以下來自次要來源，請核對原文）

**Microsoft 2013 研究**：只有 14% 的 code review 留言真的在討論 defect（Bug）。其他 86% 是風格、一致性、可讀性這些。他拿這個數字說「逐行 review 的主要效益其實不在抓 bug」——而 AI 在風格、可讀性、一致性這些事情上做得不差，所以人的 review 力氣應該集中在那 14% 真正的 defect 上。

**高嚴重性漏洞發現量**：某一季發現的高嚴重性漏洞，幾乎是前兩年加起來的兩倍。他的詮釋不是「AI 寫出更多漏洞」，而是「發現漏洞變便宜了（AI 幫你找）」——但確認漏洞是不是真的可以被利用（triage），還是要人讀程式碼才能判斷。

這兩個數字對照起來說的是同一件事：**人的注意力應該集中在 AI 做不好的地方（triage、ownership、判斷要不要 ship），而不是 AI 已經做得不差的地方（語法、格式、基本邏輯）。**

---

### 行動建議：如何決定審查深度

他提出根據風險等級分配 review 深度的框架（非原文直接引用）：

```
審查深度由兩個維度決定：
「這段程式碼在哪裡」 × 「如果它錯了會怎樣」

低風險（工具函式、無副作用的 helper）
  → 有充分測試覆蓋就夠
  → AI agent review pass 了可以直接 merge
  → 人只需 sanity check，不用逐行讀

中風險（UI 元件、第三方 API 串接）
  → 看輸入輸出邊界 + 確認錯誤處理
  → 至少要問「這段程式碼最壞情況是什麼」

高風險（auth 邏輯、金流、資料寫入、安全敏感路徑）
  → 逐字讀，要求 AI 解釋它做的每個假設
  → 用跟生成程式碼不同的 model 或不同的人來審
  → 審查者不能是生成者
```

最後一點值得特別說明。他在相關文章裡提到一個警告：

```
Goodhart's Law 應用到 AI reviewer：

如果你讓同一個 AI agent 同時寫程式碼和審程式碼，
它可能會「重寫測試來符合它寫的程式碼」，
而不是「修程式碼來符合它應該要通過的測試」。

結果：測試全綠，但驗證的是錯誤的行為。

→ 所以獨立審查（審查者和生成者要分開）非常重要。
```

---

### Comprehension Debt（理解債）

在相關文章的脈絡裡出現了一個關鍵詞：**comprehension debt（理解債）**。

> AI agent 可以比你讀程式碼更快地寫出程式碼。如果你一直讓它跑，你理解這份程式碼的速度，會越來越跟不上它生成的速度。這個差距會複利成長。

類比：就像技術債，只是累積的不是「需要重構的舊程式碼」，而是「你沒有真正讀過、但已經在 production 跑著的程式碼」。有一天你要改一段邏輯，才發現你其實不知道那段程式碼在做什麼——即使是你的 team 寫的。

**預防做法**：在 comprehension debt 累積之前，**選擇性地「主動閱讀」高風險路徑的程式碼**，哪怕那段最近沒有被改動。不是每行都讀，而是你清楚知道哪些路徑是你理解的、哪些是你不理解的。

---

### 他的時程預測

（次要來源，請核對原文）

| 領域 | 預計時間窗 |
|---|---|
| 一般 SaaS / 商業軟體 | **2 年內**幾乎不再有逐行 review |
| 企業系統 / 金融軟體 | **3–5 年**逐步轉型 |
| 航空飛控 / 高風險醫療軟體 | 繼續嚴格人工 review，沒有時間窗 |

這個分層跟他 *Brownfield Agentic Engineering* 的 blast radius 框架完全一致：越不可逆的錯誤，review 越不能省。

---

### 跟 repo 裡既有內容的連結

- **`digests/2026-10-12.md` 第 1 則**（Chrome DevTools October 2026）：`--no-javascript-evaluation` 旗標讓 AI agent 在抓 DOM 時不觸發副作用——這是「AI 在 production 執行時要有邊界」的工具層實現，跟 Osmani 說的「autonomy 要跟著 blast radius 走」是同一個思路。

- **`deep-dives/2026-08-02-followups.md` 第 4 則**（Addy Osmani LLM coding workflow）：那份導讀講的是「怎麼用 TDD 框住 AI 寫程式碼的範圍」——本文是它的後續問題：「TDD 和測試通過之後，review 這件事還需要做到什麼程度？」兩篇合起來是一套完整的 AI coding 流程觀。

---

### 最精簡版總結

整篇文章的核心只有一個切割：

> **逐行閱讀（reading）和對 ship 決定負責（owning）是兩件不同的事。**
> AI 時代讓前者越來越不必要，但後者從來不會消失。

建立信任的機制要從「有人讀過它」轉移到「這段程式碼被獨立的、可信賴的檢查機制驗過，而且有人的名字附在那個 ship 決定上」。
