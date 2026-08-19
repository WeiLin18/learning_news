# 深入追問筆記 2026-08-19

這份文件整理使用者在對話中貼出的一則外部連結、要求深入了解的內容,補上比原始素材更完整的
脈絡與例子。跟 `digests/` 不同:`digests/` 是每日固定則數的精選摘要,這份文件是**針對特定
主題的深入展開**,只在使用者主動追問時才會產生,不是每天都有。

## 目錄

1. [Uncle Bob 不讀 AI 寫的程式碼:用測試護欄取代 Code Review 的爭議](#1-uncle-bob-不讀-ai-寫的程式碼用測試護欄取代-code-review-的爭議)(追問對象:使用者直接提供的 Instagram reel,`https://www.instagram.com/reel/DbnTOD6zyes/`)

---

## 1. Uncle Bob 不讀 AI 寫的程式碼:用測試護欄取代 Code Review 的爭議

> **未能開啟原文**:Instagram reel 本身因 egress 網路政策無法存取,原始出處(Robert C. Martin
> 在 X 上的貼文串)也同樣無法直接開啟。以下內容依多組交叉搜尋(搜尋引擎回傳的文章摘要與貼文
> 引述片段)推得,直接引用的句子已盡量核對來源,但完整貼文串的前後脈絡請以原文為準。

### 這則在講什麼

2026 年 4 月起,《Clean Code》作者 Robert C. Martin(業界暱稱 Uncle Bob)在 X 上陸續發文,
表明他現在**不再逐行閱讀 AI agent 寫出來的程式碼**,改成用一整套自動化指標當關卡——只要
程式碼通過這些關卡,就直接信任它,不管有沒有人類讀過任何一個函式的內容。這個立場後來跟另一位
軟體工程界元老 Grady Booch 公開唱反調,成為近期「AI 時代 code review 還要不要做」爭論的
代表性案例。

### Uncle Bob 的方法論:用什麼取代閱讀

他在回覆 @wookash_podcast 的貼文裡直接講出核心邏輯(原文引用):

> "I don't review code written by agents. I measure things like test coverage,
> dependency structure, cyclomatic complexity, module sizes, mutation testing, etc.
> Much can be inferred about the quality of the code from those metrics. The code
> itself I leave to the AI. Humans are slow at code. To get productivity we humans
> need to disengage from code and manage from a higher level."

翻成白話:他把「人類讀程式碼」這個動作,替換成一組**自動化指標護欄**,包含:

- 測試覆蓋率(test coverage)
- 依賴結構(dependency structure)
- 循環複雜度(cyclomatic complexity)
- 模組大小(module sizes)
- 突變測試(mutation testing)
- 加上 unit test、Gherkin 驗收測試、QA 流程等一整組「extreme constraints」

邏輯是:**人類讀程式碼很慢,慢到成為生產力瓶頸**,所以與其花時間逐行審查,不如把力氣花在
把關卡設計好,讓 AI 自己在關卡裡反覆試錯,通過關卡的東西就值得信任。

值得注意的是,這不是他一時興起的立場。早在 2019 年他就講過(另一則舊貼文):「Test coverage
is a terrible management metric... but used well, it is a very valuable development
tool. Especially if you mix it with mutation testing.」——AI 生成程式碼只是把這個原本就有的
主張推到極端:徹底拿掉「人讀程式碼」這一步。

**具體怎麼量測**,他也給了實例。循環複雜度的做法很單純:所有函式壓在 CC 4 以下。更進階的是
他常提的 **CRAP 指標**(circumference/复杂度混合覆蓋率):

```
CRAP = 循環複雜度 × (1 - 測試覆蓋率)²  的概念(簡化說明,非官方公式)

高覆蓋率 → 分數被拉低(即使函式複雜,只要測試顧到,風險可控)
低覆蓋率 → 分數被拉高(複雜又沒測試的函式,直接被關卡擋下)
```

也就是「複雜但測試顧得到」放行,「複雜又沒人測」擋下——用一個數字同時懲罰「複雜」跟
「沒測試」兩件事疊加的風險。突變測試的部分,他提到自己讓 Claude 寫了一個小工具:先跑覆蓋率,
再對每一個被覆蓋到的運算子做突變,重跑對應的測試,存活下來的突變(代表測試其實沒真的驗證到
那段邏輯)會被特別列出來檢視。

### Grady Booch 的反駁

Booch 採取相反立場,他**照樣審查 agent 產出的每一段程式碼**,理由很直接:測試覆蓋率之類的
指標只能告訴你「功能大概有跑」,完全不能告訴你程式碼裡有沒有藏漏洞、有沒有留下未來會拖累
可讀性的死碼(dead code)、有沒有漏掉本來能大幅影響效能的重構機會。他的口號是 **"trust but
verify"**,並補一句「沒有任何 agent 具備人類資深工程師那種一眼看出程式碼好壞的經驗與情境
判斷力」。

| | Uncle Bob | Grady Booch |
|---|---|---|
| 是否讀 agent 寫的程式碼 | 不讀,只看指標 | 全部讀 |
| 信任來源 | 測試覆蓋率、突變測試、循環複雜度等自動化指標 | 資深工程師的經驗與情境判斷("聞得出來"好壞) |
| 對指標的態度 | 指標設計得好,就能推論出程式碼品質 | 指標只能證明「功能正常」,證明不了「沒有漏洞/死碼」 |
| 核心主張 | 人讀程式碼太慢,應該把人力放在更高層次的管理 | 指標與人工審查是互補,不能互相取代 |

兩人真正分岔的地方不是「要不要測試」——雙方都認同測試護欄很重要——而是**測試護欄能不能
完全取代人類閱讀程式碼本身**。Uncle Bob 賭的是「關卡夠嚴,就不需要再看內容」;Booch 賭的是
「有些風險(安全漏洞、可維護性、效能)天生不會反映在測試指標上,少了人讀,永遠抓不到」。

### 跟這份 repo 之前討論過的內容怎麼接

這剛好可以跟 `deep-dives/2026-08-02-followups.md` 第 4 則(Addy Osmani 的 AI coding
workflow)放在一起看,兩者其實是同一個光譜上的兩個點:Osmani 的方案是**依風險分級**——低風險
只看型別檢查+測試,高風險除了測試還要疊加兩個不同的 AI reviewer 加一個真人工程師加資安 pass;
Uncle Bob 的方案則是**不分風險等級,一律用指標關卡取代人類閱讀**,把「人要不要讀」這個決定
權整個交給關卡設計本身。實務上,Osmani 那種分級模式目前看起來是業界比較主流、也比較容易說服
團隊採用的折衷方案,Uncle Bob 的極端立場更像是他個人多年對「test coverage 該怎麼用」主張的
延伸實驗,還在被業界公開辯論、尚未形成共識。

### 這對一般團隊的意義

如果要從這場爭論裡抽一句能直接用的建議:**先把測試護欄(覆蓋率門檻 + 突變測試)當成必要
但不是充分的條件**。護欄能擋掉「明顯不能動」的程式碼,但護欄本身不會告訴你「這段程式碼是不是
埋了一個難以察覺的安全漏洞」——那部分目前仍然只有經驗豐富的人(或另一個獨立的 AI reviewer)
看過才有機會抓到。全面採用 Uncle Bob 那種「完全不讀」的模式之前,至少對高風險程式碼(權限、
金流、資安相關)保留人工或多模型審查這一關,會是比較穩健的折衷。
