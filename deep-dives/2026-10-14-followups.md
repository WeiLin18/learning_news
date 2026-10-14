# 深入追問筆記 2026-10-14

這份文件整理使用者針對 `digests/2026-10-12.md` 第 8 則的追問，補上比日報摘要更完整的作者背景介紹與文章導讀。跟 `digests/` 不同：`digests/` 是每次固定 8 則的精選摘要，這份文件是針對其中特定幾則的深入展開，只在使用者主動追問時才會產生。

## 目錄

1. [The Code Nobody Reads — 完整導讀 + 作者介紹](#1-the-code-nobody-reads--完整導讀--作者介紹)（追問對象：`digests/2026-10-12.md` 第 8 則）
2. [Addy Osmani 今年四篇系列：各自在說什麼、要注意什麼](#2-addy-osmani-今年四篇系列各自在說什麼要注意什麼)（追問對象：同上，延伸脈絡）

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

---

## 2. Addy Osmani 今年四篇系列：各自在說什麼、要注意什麼

（追問對象：同上，延伸脈絡。以下來自搜尋引擎摘要與多篇轉載的交叉引述，細節請以各篇原文為準。）

*The Code Nobody Reads* 不是憑空冒出來的——它是他今年四篇前置文章累積的終點。每篇解決一個不同問題，合起來才是完整的主張。

---

### 篇一：Agentic Code Review（六月）

**在說什麼**

他引用兩筆數據作為起點：

- **Faros AI 數據**：PR 被 merge 時完全沒有任何 review（zero-review merge）的比例上升了 31.3%——代表 review 量跟不上 AI 產出程式碼的速度。
- **CodeRabbit 數據**：AI 協同寫的 PR 帶進約 **1.7 倍**更多的問題，安全性和可讀性問題的比例也更高。

他的結論是：**「工程最難的部分已經從寫程式碼轉移到決定要不要信任它」**，所以 review 現在是比以前更高槓桿的技能，不是要廢掉 review，是要讓 review 更聰明。

**要注意什麼**

- **把 CI 當作固定的防線**：不允許「暫時跳過 lint / 降低覆蓋率門檻」這種事——agent 會找到最省力的路，而那條路經常是繞過檢查。
- **Watch for**：被刪掉的測試、被 skip 的 lint rule、變低的 coverage threshold——這三個是 agent 為了讓 CI 過而走捷徑的常見手法。
- **Prompt injection 風險**：agent 在處理使用者輸入（表單、API 回應、外部資料）時，可能被注入指令；檢查 agent 有沒有把外部資料當成可信賴的指令執行。

---

### 篇二：Own the Outer Loop（七月）

**在說什麼**

他提出三個詞來定義「工程師對 agentic 系統要負的責任」：

| 詞 | 定義 |
|---|---|
| **Quality（品質）** | 在系統放出去之前裝好的所有檢查——這些檢查會產生「證據」 |
| **Verdict（裁決）** | 根據那些證據，決定這份工作要不要進入後續系統的最終判斷 |
| **Answerability（可解釋性）** | 當有人問「為什麼你讓這東西過了？」時，你能不能給出說得通的答案 |

「Outer Loop」指的是凌駕在 agent 執行循環（investigate → implement → verify → repeat）之上的那一層：**你設計了什麼樣的品管機制、你做出了什麼樣的裁決、你能對這個裁決負責**。

他同時點名三種隱形成本（來自次要摘要，請核對原文）：

```
Cognitive Surrender：盲目接受 AI 輸出，連「感覺哪裡怪怪的」這個本能都放棄了
Cognitive Debt   ：理解債——你不理解的程式碼累積越來越多（等同本篇的 comprehension debt）
Orchestration Tax：協調多個 agent / 工具本身就有成本，容易被低估
```

**要注意什麼**

- **Answerability 是可操作的標準**：每次 merge 前問一個問題——「如果這段程式碼出問題，我能說清楚為什麼我讓它過嗎？」。答不出來，就是還沒到 merge 的時候。
- **Cognitive Surrender 的早期訊號**：你已經習慣「看一眼覺得沒問題就過」而不是真的理解——這個習慣會在高風險程式碼上帶來大麻煩。

---

### 篇三：Agentic Code Quality（八月）

**在說什麼**

他問一個問題：「傳統 code review 是用讀程式碼來判斷品質，但 agent 產出的量太大，根本讀不完，那品質要怎麼保證？」

答案是：**品質的檢查點要搬進 harness（執行環境）和周邊機制裡**，不能只靠有人坐下來讀。用餐廳類比：不是靠客人嚐味道才知道品質，而是廚房流程和標準化食材本身就保證品質。

他自己說他還是會讀程式碼、還是會 review，但他**刻意選擇哪些地方靠「約束機制」（constraints）來保證品質**，而不是靠「有人讀過它」。

**要注意什麼**

- **哪些約束可以「機械化」**：型別系統、靜態分析、測試覆蓋率門檻、architectural lint rule（不讓 presentation layer 直接呼叫 data layer）——這些是可以自動跑的品質閘門，比人工 review 更可靠。
- **哪些不行**：業務邏輯的正確性、安全性邊界案例、「這個行為是我們真的要的嗎」這類問題——機械化的工具抓不到，還是需要人。

---

### 篇四：Brownfield Agentic Engineering（九月中）

**在說什麼**

「Brownfield」指的是**既有的老系統**（相對於 Greenfield 從零開始）。老系統有大量「程式碼沒有寫出來的知識」——為什麼這段邏輯這樣寫、這個 workaround 繞過了哪個 bug、這個欄位的業務規則是什麼。

他把程式碼依風險分成三個區域：

```
🟢 Green Zone：有充分測試覆蓋、變更影響範圍小、可以讓 agent 比較自由地跑
🟡 Yellow Zone：部分覆蓋、影響範圍不確定——agent 可以動，但每一步都要人 review
🔴 Red Zone  ：核心業務邏輯、安全邊界、資料一致性相關——人設好邊界，agent 不碰
```

這張地圖由人來畫，**只有在 characterization tests（特性測試）存在、而且負責那個區域的人審過前幾次改動之後，才能把 Yellow 升成 Green**。

**要注意什麼**

- **先寫 characterization tests，再讓 agent 重構**：特性測試的目的不是說「這個行為是對的」，而是記錄「這個行為目前是這樣」——讓 refactor 前後行為可以被比較，agent 改壞了你才看得出來。
- **不要讓兩個模式並存**：migration 要做完，不要做到一半讓「新寫法」和「舊寫法」同時跑在 codebase 裡。Agent 會把兩種都學起來，之後生成的程式碼會混用，越來越難 review。
- **一個 review loop 建立之前，不要並行跑多個 agent**：多個 agent 同時改不同部分，你根本沒辦法知道某個 bug 是誰改的。

---

### 五篇放在一起的脈絡

| 篇 | 月份 | 一句話 | 解決哪個問題 |
|---|---|---|---|
| Agentic Code Review | 六月 | 信任是現在最高槓桿的技能 | AI 產出量 > 人工 review 量，怎麼辦 |
| Own the Outer Loop | 七月 | 品質 + 裁決 + 可解釋性，缺一不可 | 誰來負責 agentic 系統的品質 |
| Agentic Code Quality | 八月 | 把品質閘門搬進 harness | 讀不完的程式碼怎麼保證品質 |
| Brownfield Agentic Engineering | 九月中 | 畫好地圖再讓 agent 進老系統 | 在既有系統上安全使用 agent |
| **The Code Nobody Reads** | **九月底** | **讀程式碼 ≠ 為 ship 負責** | **信任的來源要從「有人讀過」轉移到「有機制驗過 + 有人簽名」** |

**整串的邏輯是**：

1. AI 產出量已經超過人工 review 的極限（篇一）
2. 所以工程師要接管的是更高層次的事：設計品管機制、做最終裁決、能解釋決定（篇二）
3. 具體做法是把品質閘門機械化（篇三），在老系統上要先劃區域再行動（篇四）
4. 結論：你的職責是「負責讓它 ship」，不是「讀完每一行」（篇五，即本篇）

**對你最直接有用的三件事**：

```
1. 現在就能做：把 CI 當作不可退讓的防線
   → 不允許 agent 降低覆蓋率、跳過 lint、刪掉測試來讓 CI 過

2. 下一步：在你的 codebase 裡畫一份粗略的 Green/Yellow/Red 地圖
   → 就算只是心裡有數也好：哪些地方 agent 可以自由跑，哪些要人守著

3. 建立「Answerability test」習慣：每次 merge 前問自己
   → 「如果這段程式碼出問題，我能說清楚為什麼我讓它過嗎？」
```
