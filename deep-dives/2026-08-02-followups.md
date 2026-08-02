# 深入追問筆記 2026-08-02

這份文件整理使用者在對話中針對日報內容追問、要求「多說明」的部分,補上比日報摘要更完整的解釋與例子。跟 `digests/` 不同：`digests/` 是每日固定 15 則的精選摘要,這份文件是**針對其中特定幾則的深入展開**,只在使用者主動追問時才會產生,不是每天都有。

## 目錄

1. [TanStack Start vs Next.js vs Remix 架構差異](#1-tanstack-start-vs-nextjs-vs-remix-架構差異)（追問對象：`digests/2026-08-02.md` 第 4 則）
2. [Vercel AI SDK 7 詳細功能與簡單例子](#2-vercel-ai-sdk-7-詳細功能與簡單例子)（追問對象：`digests/2026-08-02.md` 第 10 則）
3. [Storybook 10 詳細功能與簡單例子](#3-storybook-10-詳細功能與簡單例子)（追問對象：`digests/2026-08-02.md` 第 14 則）
4. [Addy Osmani LLM coding workflow 完整拆解與具體執行步驟](#4-addy-osmani-llm-coding-workflow-完整拆解與具體執行步驟)（追問對象：`digests/2026-07-31-2.md` 第 6 則）

---

## 1. TanStack Start vs Next.js vs Remix 架構差異

### 核心設計哲學

- **Next.js**：Vercel 主導，以「框架幫你決定很多事」為哲學，App Router + React Server Components 是預設心智模型，生態系最大、最成熟。
- **Remix / React Router v7**：強調「貼近 Web 標準」的 loader/action 心智模型（用 `<form>`、`fetch` 這些瀏覽器原生概念，不特別發明新抽象），Remix 已併入 React Router v7。
- **TanStack Start**：由 Tanner Linsley（TanStack 系列作者）主導，核心訴求是「不用重量級抽象、但要有端到端的型別安全」，把 TanStack Router 的型別系統延伸到全端。

### 路由與型別安全（最大差異點）

| | Next.js | Remix/React Router | TanStack Start |
|---|---|---|---|
| 路由定義 | 檔案慣例（`page.tsx`、`[postId]` 資料夾） | 檔案慣例 + 巢狀路由設定 | 檔案慣例（`index.tsx`、`$postId`），但路由是**函式**，編譯期就檢查路由形狀 |
| 型別安全程度 | 表面性——主要靠 IDE plugin 提示連結，不是編譯期保證 | 中等，loader/action 型別靠約定 | **端到端強制型別**：路由參數、search params、loader 回傳值全部編譯期檢查 |
| 巢狀路由 | 支援（layout.tsx） | 原生強項，Remix 從一開始就是為巢狀路由設計 | 一級支援，概念接近 Remix |

三者裡最本質的差異：TanStack Start 把「路由本身要不要型別安全」當成第一原則，Next.js 是事後補的體驗層，不是編譯期保證。

### 資料載入 / Server Functions

- **Next.js**：Server Actions + Server Components 資料獲取，資料抓取邏輯常常直接寫在 Server Component 裡。
- **Remix**：`loader`（讀）/`action`（寫）分離，每個路由自己定義，是最早把這個模式帶紅的框架。
- **TanStack Start**：同樣是 loader 概念（跟 Remix 很像），但額外提供**isomorphic server functions**——同一支函式可以在伺服器跑，也可以直接從瀏覽器呼叫，TypeScript 型別會自動對齊兩端，不用手動維護一份 API 契約。

### Server Components（RSC）支援

- **Next.js**：RSC 是預設路徑，最成熟。
- **Remix**：目前不是 RSC 為主的模型。
- **TanStack Start**：目前**還沒有** RSC，官方路線圖上是「未來會支援」，現階段是 SPA + SSR + streaming 的組合，這是它目前相對 Next.js 最大的功能落差。

### 底層打包工具與部署

- **Next.js**：Turbopack（逐步取代 Webpack），部署到非 Vercel 平台或純 edge 環境常需要額外設定。
- **Remix**：原生對 Cloudflare Workers 等 edge 平台支援度高。
- **TanStack Start**：基於 Vite，開發體驗上啟動快、HMR 快、資源占用低；1.0 之後也做到了 production-grade 的 Cloudflare Workers 支援。

### 該怎麼選（業界現在的共識）

- **生態系/招募考量優先 → Next.js**：社群最大、職缺最多、第三方整合最深，是保守選擇。
- **想要貼近 Web 標準、輕量、但不用 Next.js 全家桶 → Remix/React Router v7**。
- **在意編譯期型別安全、caching 設計、心智模型更簡單，團隊願意當早期採用者 → TanStack Start**：目前業界普遍定位是「值得關注但仍屬早期採用」，主要風險是還沒有 RSC、生態系比另兩者薄。

真正分岔的地方是**型別安全的嚴格程度**（TanStack Start 最嚴）跟**RSC 成熟度**（Next.js 最成熟，TanStack Start 目前沒有）。

---

## 2. Vercel AI SDK 7 詳細功能與簡單例子

### 背景轉變

AI SDK 6 及以前主要解決「在 React app 裡串流 LLM 回應」（`useChat`、`streamText`）。AI SDK 7 的重心轉向「讓 agent 在正式環境長期穩定跑」，三個新元件分別解決三個不同痛點。

### `HarnessAgent` — 統一介面呼叫現成的 agent harness

類比：像叫外送用同一個 App，不用每家餐廳分別下載一個 App。過去串接 Claude Code、Codex 各要學一套不同的權限/session 管理方式，現在只是換一個參數：

```ts
const agent = new HarnessAgent({ harness: 'claude-code' });
// 換成 codex,程式碼幾乎不用改
const agent2 = new HarnessAgent({ harness: 'codex' });
```

harness 本身負責管理 skills、sandbox、session、權限流程、上下文壓縮（compaction）、子 agent。

### `WorkflowAgent` — 解決「agent 中途掛掉怎麼辦」

類比：像網購退款要等審核，審核期間你可以關掉 App，審核過了它自己繼續。情境：agent 要刪除一筆重要資料前，必須等主管按「核准」，可能要等 3 天，期間伺服器也重啟了幾次：

```ts
const agent = new WorkflowAgent({
  tools: { deleteRecord },
  onApprovalNeeded: async () => waitForHumanApproval(), // 這裡可以「卡」3 天沒關係
});
```

沒有 WorkflowAgent 的話，伺服器一重啟，「等核准」這件事就會忘記，得整個流程重跑一次。

### MCP Apps 的 model-visible vs app-only

類比：像菜單（客人看）跟出餐單（廚房內部用）。客人只需要看到「宮保雞丁 $180」，不需要看到廚房內部的食材庫存編號：

```ts
const searchTool = { name: 'search', visibleToModel: true };   // AI 看得到,會拿來決策
const chartTool  = { name: 'renderChart', appOnly: true };     // 只是畫個圖給使用者看,AI 不用知道
```

如果所有東西都給 AI 看，不只浪費 token，內部資料也可能不小心洩漏給 AI。再加上 HMAC 簽名的工具核准機制，防止有人偽造「使用者已核准」這個訊號去執行高風險操作。

---

## 3. Storybook 10 詳細功能與簡單例子

### Module Automocking（`sb.mock`）

跟 Vitest 團隊合作做出來的，靈感來自 `vi.mock`，相容 Vite 跟 Webpack，dev 模式跟正式 build 都能用。

類比：像展示間的樣品車，儀表盤數字是假資料，不用真的發動引擎。

```ts
// Button.stories.tsx
import { sb } from 'storybook/test';

sb.mock('~/api/user', () => ({
  fetchUser: () => ({ name: '測試使用者', avatar: '/mock.png' }),
}));
```

不用 API 真的回應，畫面照樣能開發、能測試。

### Typesafe CSF Factories（從 Experimental 升級到 Preview）

CSF 3（目前主流寫法）用 plain object export，型別推導有限。CSF Factories 改用 `meta()` 產生的 factory 函式定義 story，型別完全跟元件 props 對齊。

類比：像填表單時欄位打錯字馬上被擋下來，而不是送出後才被退件。

```ts
const meta = config({ component: Button });

// 假設 Button 元件根本沒有叫 lodingg 的 prop(打錯字)
export const Bad = meta.story({ args: { lodingg: true } });
// ❌ 寫的時候編輯器立刻紅字提示,不用等執行才發現

export const Good = meta.story({ args: { loading: true } });
// ✅ 正確拼字,而且型別是從 Button 元件自動推導出來的
```

另新增 `.test` method，可以直接把測試邏輯掛在 story 上，這種測試用 story 會自動從側邊欄（給設計師/PM 看的 UI）隱藏：

```ts
export const Loading = meta.story({
  args: { loading: true },
  test: async ({ canvas }) => {
    await expect(canvas.getByRole('button')).toBeDisabled();
  },
});
```

**限制**：CSF Factories 目前只支援 React，Vue/Angular/Web Components 版本官方說「10.x 系列會補上」，還沒到。

### 跟業界趨勢的關聯

Storybook 的 automocking 跟 CSF Factories，本質上都是在把「元件測試」跟「元件文件/展示」融合成同一份程式碼——這跟 `digests/2026-08-02.md` 第 11 則 Playwright 1.62 的「story + gallery」元件測試模型，是同一個業界趨勢：用同一份 story 定義同時餵給「文件展示」跟「自動化測試」兩邊用，不用維護兩套。

---

## 4. Addy Osmani LLM coding workflow 完整拆解與具體執行步驟

原文大約 2025 年 12 月底發布（標題取「going into 2026」），完整脈絡比日報摘要豐富不少。

### 完整工作流程

**1. 先寫規格，不要先寫程式碼**
跳過這一步直接讓 AI 寫，結果來回修改浪費更多時間。做法是先跟 AI 一起把「要做什麼」寫成一份清楚的 spec/plan。

**2. 把 spec 拆成一份「prompt plan」檔案**
產生一個結構化檔案，裡面是**針對每個任務的一連串 prompt**，讓 Cursor 這類工具可以照順序執行：

```markdown
## prompt-plan.md
### Step 1
"實作 `validateEmail(input: string): boolean`,只處理格式驗證,不用管重複信箱檢查"
### Step 2
"幫 Step 1 的函式寫 5 個邊界案例的測試(空字串、缺少 @、多個 @...)"
### Step 3
"實作 checkEmailExists(email) 呼叫現有的 UserRepository,不要改動 UserRepository 本身"
```

**3. 每一步都搭配 TDD**
先有測試，AI 才動手改程式碼：

```
你:「這是 validateEmail 的測試,目前是紅的,幫我用最小改動讓它變綠」
AI:(只寫剛好讓測試通過的程式碼,不會順手多加其他功能)
你:review diff → 通過才進 Step 2
```

**4. Review 完再進下一步，不累積大顆未審查的變更**
批次越小，出錯時要回溯的範圍越小。

### 品質關卡（依風險決定審查力氣，這部分是延伸自他另一篇《Agentic Code Review》）

| 風險等級 | 審查方式 |
|---|---|
| 低風險(小型 UI 調整、文案修正) | 型別檢查 + 測試通過就夠,人只抽查 |
| 中風險(一般功能開發) | 測試 + 一個 AI reviewer 過一輪 |
| 高風險(權限、金流、安全相關) | 型別 + 測試 + **兩個不同的 AI reviewer** + 一個真正負責這個系統的工程師 + 資安 pass |

**多模型審查（multi-model review）**：同一個 model 生成的程式碼，再拿同一個 model 去審，盲點會一樣；換一個不同廠商/不同訓練方式的 model 來審，抓到的問題種類會不一樣。

**測試覆蓋率門檻**：建議把覆蓋率 >70% 當成 AI 產出程式碼能不能合併的硬性門檻。

### 具體照做步驟（把上面理論變成實際操作,以「註冊表單 email 驗證」為例）

**第 0 步**：把任務拆成清單（拆到「單個函式/單個元件」的粒度就夠，不用拆到誇張細）：
```
1. validateEmail() 格式驗證函式
2. checkEmailExists() 檢查信箱是否已註冊
3. 表單 UI 元件(輸入框 + 錯誤訊息顯示)
4. 串接兩個函式到表單的 submit 流程
```

**第 1 步**：存成 `docs/prompt-plan.md`,把每一步寫成「等一下要跟 AI 說的話」（見上方 prompt-plan 範例)。好處：下次做類似任務可以重複用這份模板。

**第 2 步**：先寫測試,再叫 AI 動手：
```
你(對 AI 說):「這是 validateEmail 的測試檔案(貼上 Step 2 產出的測試,目前全部是紅的)。
             請寫最小的 validateEmail 實作讓這些測試通過,不要加測試沒要求的功能。」
```

**第 3 步**：每完成一步就 review：
1. AI 給你 diff → 看 `git diff` 或 IDE 的變更視圖
2. 只看**這一步**改了什麼(範圍小,通常 1 分鐘內看得完)
3. 測試綠了 + diff 合理 → commit,進下一步
4. 測試沒過或 diff 看起來怪 → 當場退回去,不帶著問題進下一步

**第 4 步**：依風險決定要不要多一層 AI 審查（表單文案/CSS 調整自己看一眼就好；牽涉密碼/金流/權限的部分,測試過後**開一個全新對話**,貼上程式碼問另一個 model「你是資安審查員,幫我找這段程式碼的漏洞」——全新對話很重要,同一個對話裡它記得自己剛才怎麼寫的,審查會偏向幫自己背書）。

**第 5 步**：在 CI 設一個測試覆蓋率的硬門檻：
```ts
// vitest.config.ts
coverage: { thresholds: { lines: 70, functions: 70 } }
```
低於 70% 直接讓 CI fail,把「人工要求覆蓋率」變成自動化關卡。

**最精簡版總結**：先寫測試 → 每次只請 AI 做能讓一個測試變綠的最小改動 → 看過 diff 才進下一步。其他(prompt-plan 檔案、多模型審查、覆蓋率門檻)都是這個核心習慣的延伸,可以之後再慢慢加。
