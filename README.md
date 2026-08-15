# learning_news

每隔約兩天蒐集一次的 Web 前端相關新聞與資訊摘要（一週約 3–4 次），由 agent 產生並開 PR。

> 個人版本，參考自 [BillChai/learning_news](https://github.com/BillChai/learning_news)。

## 結構

- `AGENTS.md` — 這個 repo 的**唯一事實來源**：完整的選文/開 PR 指令，廠商中立，任何具備網路
  搜尋與 GitHub 讀寫能力的 AI agent 都能照它獨立執行。
- `digests/YYYY-MM-DD.md` — 每次的 8 則精選新聞/資訊摘要。
- `deep-dives/YYYY-MM-DD-followups.md` — 針對日報中特定幾則的深入導讀（類比、程式碼範例、
  對照表、可照做步驟、跨則統整）。只在使用者主動追問時產生，不是每天都有。
- `preferences.md` — 累積記錄使用者對過去內容的喜歡/不喜歡回饋，用來調整之後每次挑選的主題與來源權重。
- `.claude/skills/deep-dive/` — 產生上述導讀的 Claude Code skill；在對話裡說「第 4 則多說明一點」
  這類話就會觸發。日報本身的流程仍由廠商中立的 `AGENTS.md` 定義，不依賴這個 skill。

## 流程

1. 讀取 `preferences.md` 了解目前偏好。
2. 依 `AGENTS.md` 的規則搜尋近期 Web 前端相關新聞，挑選 8 則，寫成 `digests/YYYY-MM-DD.md`。
3. 開一個 PR（`digest-YYYY-MM-DD` -> `main`），內容為當天的摘要檔案。
4. 使用者在對話中給回饋後，偏好會被追加寫入 `preferences.md`（同一個 PR 或後續 commit）。
5. 使用者自行決定要 merge 還是 close 該 PR——**這兩個動作有不同意義**，見下。

### merge 與 close 的差別

PR 的最終狀態就是「這批內容有沒有真的送達」的訊號，下次執行時會據此調整（規則見 `AGENTS.md` Step 6）：

| 你的動作 | 代表的意思 | 下次執行的行為 |
|---|---|---|
| **merge** | 收下了 | 這些文章**永不再出現** |
| **close**（不 merge） | 當時沒時間看 | 會從裡面挑**最多 2 則**耐放的精華，帶 `[補收]` 標籤放進新的一份 |
| **開著不動** | 還沒決定 | 不會重複選到裡面的內容 |

所以沒時間看的時候直接 close 就好，不用擔心好內容就這樣消失。若某一批是因為**品質不好或主題
不想看**才關掉的，在 close 時留一則留言說明，下次就不會把它補收回來。

## 如何用任何 AI 執行

這個 repo 是自足的：`git clone` 之後，把下面這句話貼給任何一個具備「網路搜尋」與「對這個
repo 的 GitHub 讀寫/開 PR」能力的 AI（Claude、Gemini、ChatGPT/Codex 等皆可），就能重現同樣的
行為：

> 請 clone 或開啟 GitHub repo `WeiLin18/learning_news`，讀取根目錄的 `AGENTS.md` 並完整照做，
> 今天日期是 YYYY-MM-DD。

**注意事項**：

- 每個 AI 帳號**第一次**使用前，需要人工在該 App/服務的設定裡做一次性 GitHub 授權（例如
  Claude 的 Connectors、ChatGPT Codex 的 repo 綁定、Jules 的 repo 選擇）。這一步無法寫進上面
  那句 kickoff prompt 裡，是純人工設定，且是「每個帳號一次」而不是「每次執行都要做」。
- 避免同一天讓兩個 agent 各自跑一次——`AGENTS.md` 裡雖然有處理分支撞名的規則（自動改用
  `-2`、`-3` 尾碼），但 `preferences.md` 的回饋追加順序仍可能因此變得混亂，建議一天只執行一次。
