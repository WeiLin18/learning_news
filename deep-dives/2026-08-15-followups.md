# 深入追問筆記 2026-08-15

這份文件整理使用者針對日報內容（或直接提供的文章網址）追問、要求「深入講講」的部分，補上比日報摘要更完整的解釋、可照做的步驟與實際指令。跟 `digests/` 不同：`digests/` 是每次執行固定 8 則的精選摘要，這份文件是**針對特定主題的深入展開**，只在使用者主動追問時才會產生。

## 目錄

1. [把一個巨大的 AI 生成 PR 拆成可審查的 stack](#1-把一個巨大的-ai-生成-pr-拆成可審查的-stack)（追問對象：使用者直接提供，未收錄於既有日報）

---

## 1. 把一個巨大的 AI 生成 PR 拆成可審查的 stack

原文：GitHub Blog，作者 Julia Muiruri，2026-08-04 發布。這篇是配合 GitHub **stacked pull requests 於 2026-07-30 進入 public preview** 而寫的實務指南——所以它同時是產品文也是方法論文，讀的時候要把兩者分開看。

### 這在解決什麼：一個很具體的失敗場景

coding agent 現在有能力一口氣把「商品搜尋功能」整包做完：資料模型、seed 資料、API route、驗證、前端串接、UI 元件、各種狀態處理——原文舉的例子是**超過 1,500 行的單一 PR**。

問題不是 AI 寫得不好，是**這種 PR 沒有人能真的審查**。這正好接上 `digests/2026-08-07.md` 第 6 則 Addy Osmani 講的那個落差：96% 的人不信任 AI 產生的程式碼，但只有 48% 真的會去驗證。1,500 行的 diff 就是那 48% 消失的地方——不是大家不想看，是打開來根本無從看起，最後變成「測試綠了就按 Approve」。

**生活化類比**：這像 IKEA 家具說明書。

- 一次攤開 1,500 個零件跟一張總圖 →「看起來很完整」，但你根本不知道從哪片開始檢查，裝反了也要到最後才發現。
- 拆成「第 1 步組框架 → 第 2 步裝抽屜滑軌 → 第 3 步上面板」→ 每一步做完都能停下來確認，錯了只要退一步。

stacked PR 就是第二種。差別不在總工作量，在**驗收點的密度**。

### 原文的骨架：把功能切成有依賴順序的層

原文的商品搜尋範例被切成四層，每層一個分支、一個 PR，下層是上層的 base：

| 層 | 分支 | 這層只負責 | base |
|---|---|---|---|
| L1 | `feat/catalog-data` | 型別化的商品目錄、seed 資料、驗證、資料存取模組 | `main` |
| L2 | `feat/search-api` | `/api/products/search` 端點 | L1 |
| L3 | `feat/chat-grounding` | 聊天流程去呼叫真實商品 API | L2 |
| L4 | `feat/grounded-ui` | 商品引用卡片與 UI 狀態 | L3 |

切分的判準只有一條：**每層只處理一個關注點，而且小到能整個放進 reviewer 腦袋裡**。

順帶一個實務好處是**審查人可以分開派**——L1 給資料 owner、L4 給 UI owner，不用四個領域的人全都被迫看完 1,500 行。

### 具體照做步驟

前置：GitHub CLI 2.0 以上，安裝擴充套件。

```bash
gh extension install github/gh-stack
```

原文也提到可以裝成 Copilot 的 skill（`gh skill install github/gh-stack`），讓 coding agent 自己照這套流程做——這是這篇文章真正的主張：**stack 不只是給人用的審查工具，是拿來「教」agent 怎麼分解任務的規格**。

**第 1 步：先決定 stack base，再開工。**

```bash
gh stack init -b main
```

這一步不能事後補。原文明確提到一個容易踩的雷：**所有 PR 的 CI 都是對「stack base」跑，不是對彼此跑**。base 選錯，整疊的 CI 訊號就都是對不上的。

**第 2 步：由下往上，一層一個分支。**

```bash
# 站在 main 上先做最底層：資料模型 + seed + 驗證
gh stack add feat/catalog-data -A -m "L1: 型別化商品目錄與資料存取"

# 做完 L1 才往上疊 L2，此時 L2 的 base 自動是 L1
gh stack add feat/search-api -A -m "L2: /api/products/search 端點"

gh stack add feat/chat-grounding -A -m "L3: 聊天流程串接真實商品資料"
gh stack add feat/grounded-ui   -A -m "L4: 商品引用卡片與 UI 狀態"
```

`-A` 是把所有變更 stage 起來一起提交（跟 `-u` 互斥，`-u` 只收已追蹤檔案）。隨時可以用 `gh stack view` 看目前這疊長什麼樣、各層對應哪個 PR。

**第 3 步：推上去、開 PR。**

```bash
gh stack push      # 推送所有還沒 merge 的分支
gh stack submit    # 建立/更新 PR，並在 GitHub 上串成一疊
```

`gh stack submit --open` 會直接開成 ready for review。開完之後 GitHub 網頁上每個 PR 會出現一張 **stack map**，可以在同一疊的 PR 之間跳。

**第 4 步：審查順序是「先上後下，再由下往上看」。**

這是原文對 reviewer 最有價值的一句：

1. 先**由上往下**掃一遍，理解這疊最後要達成什麼（看目的地）。
2. 再**由下往上**逐層審 L1 → L4（看實作）。

理由很直觀：L4 的 UI 之所以長那樣，是因為 L2 的 API 回傳那個形狀；不先知道終點，你會在 L1 就開始糾結「為什麼欄位要這樣設計」。

**第 5 步：收到 review 意見後，改下層然後往上同步。**

```bash
gh stack rebase   # 串聯 rebase，改動從該層一路往上帶
gh stack sync     # 一次做完 fetch + 串聯 rebase + push + 同步 PR 狀態
```

⚠️ **這裡有個安全性的坑值得單獨標出來**：GitHub 網頁上有一鍵 rebase 按鈕，但原文明講——網頁 rebase 會**把 committer 換成按按鈕的那個人，而且產生的 commit 沒有簽章**。如果你的 branch protection 要求 signed commits，那一鍵會安靜地把規則弄壞。**能用本機 `gh stack rebase` 就不要用網頁按鈕。**

**第 6 步：合併。**

```bash
gh stack merge --merge-method squash
```

依官方文件，`gh stack merge` 是**全有全無**（all-or-nothing）：整疊一起合，而且不能繞過既有的 merge 規則。

### 依風險分級：什麼時候值得做全套

坦白說，全都做的建議沒人會照做。我的分法是：

| 情境 | 建議做法 |
|---|---|
| 100–300 行、單一關注點（改個文案、加個 lint 規則） | **不要用 stack**，一個普通 PR 就好。疊起來的維護成本大於收益。 |
| 500 行上下、跨 1–2 層（例如加一個 API + 前端接上） | 拆成 2 層就夠：資料/API 一層、UI 一層。 |
| 1,000 行以上、AI 一口氣生成、跨資料模型到 UI | **全套 stack**：分層 + 分派不同 reviewer + 由下往上審。這是原文設想的主場景。 |
| 牽涉權限、金流、schema migration | 全套 stack，而且**底層那個 PR 要單獨慢審**——把最不可逆的東西放最下面，它一旦 merge 就很難退。 |

### 這篇沒說的：外部觀點與已知爭議

原文是 GitHub 自家產品文，所以下面這些它不會寫，但選型時該知道（以下依外部報導與社群討論整理，非原文內容）：

- **這個模式不是 GitHub 發明的。** Graphite（2025 年 12 月被 Cursor 收購）、Meta 的 ghstack、`git spr`、git-spice、Sapling、Aviator 都做同一件事很多年了。GitHub 的差異是**原生**——不用第三方帳號、和 CLI／網頁／手機 App／Copilot 打通。
- **squash-merge 相容性與串聯 rebase 衝突，是這類工具至今公認沒有完全解掉的問題。** 連做了多年的 Graphite 也還在處理。層數越多、下層改得越兇，`gh stack sync` 要你手動解的衝突就越多。官方文件也註明 `gh stack sync` 在**非互動式終端**遇到分歧會直接中止（例如在 CI 裡跑）。
- **有一派根本質疑這個工作流。** 常見的反駁大致是：改動如果彼此獨立，那本來就該開成各自獨立的 PR；如果彼此相依，分開審查其實看不出全貌。我覺得這個批評有道理但過頭了——它假設 reviewer 有無限注意力。實務上「看不到全貌的四個小 diff」仍然比「看得到全貌但沒人真的看」的一個大 diff 好。
- **public preview 狀態**：功能與指令細節可能還會變，現在把它寫進團隊的硬性流程規範要保留調整空間。

### 最精簡版總結

**在讓 agent 動手之前，先跟它講好這個功能分幾層、每層的邊界在哪；然後由下往上一層一個 PR。**

其他都是延伸——`gh stack` 的十幾個指令、stack map、分派不同 reviewer、CI base 設定，全部是在服務這一件事。工具沒到位也能先做：手動開四個分支、把 base 一個指到一個，同樣拿得到八成的好處。

真正的轉變是心態上的：**過去是「寫完再想怎麼拆」，現在是「動工前就決定怎麼拆」。** 這跟 `digests/2026-07-31-2.md` 第 6 則、以及 `deep-dives/2026-08-02-followups.md` 第 4 節整理的 Addy Osmani 工作流（先寫 prompt plan → 每步最小改動 → review 過才進下一步）是同一個原則，只是那邊的顆粒度是「一個函式」，這邊放大到「一個 PR」。搭配 `digests/2026-08-12.md` 第 5 則整理的「AI 產出程式碼常見的 4 類 bug」當各層的檢查清單，就是一套完整的 AI 產出把關流程。

📖 [GitHub Blog](https://github.blog/engineering/turn-one-giant-ai-generated-pull-request-to-a-reviewable-stack/)｜2026-08-04｜作者 Julia Muiruri
📖 指令細節核對自 [GitHub Docs — Stacked pull requests CLI commands](https://docs.github.com/en/pull-requests/reference/stacked-prs-cli-commands)
📖 功能狀態核對自 [GitHub Changelog — Stacked pull requests are now in public preview](https://github.blog/changelog/2026-07-30-stacked-pull-requests-are-now-in-public-preview/)（2026-07-30）
