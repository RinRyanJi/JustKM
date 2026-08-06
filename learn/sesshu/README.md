# Sesshu 是什麼

> 回到 [知識區入口](_index.md)

Sesshu（摂取，日文「攝取」）是一個 **Python hook 套件**，
安裝到 Claude Code、Codex CLI、Gemini CLI 或 GitHub Copilot CLI 之後，
會在你的 coding session 變長時，把對話歷史壓縮成一份結構化摘要，
讓你可以安心 `/clear` 重開，而不會丟失上下文。

**來源**：`README.md:7`、`pyproject.toml:8`

---

## 1. 它解決什麼問題

### 問題：長 session 的 input token 是平方成長

每一輪對話，coding agent 都會把**整段歷史**當成 input 重新送一次。
一個開到 N 輪的 session，累積 input token 大約是三角形和：

```
Total input ≈ N × (peak_context / 2)
```

來源 `README.md:113-119` 的例子：一個跑到 190K context、40 輪的 session，
累積約 **3.8M input tokens**，而且絕大部分是同一段歷史被反覆送出。

### 第二個成本點：compaction 事件

當 runtime 的 auto-compaction 在 context 快滿（約 95%）時觸發，
**整段 pre-compaction context 會被 frontier model 掃過一次來產生摘要**，
同時 prompt cache 被打斷，接下來幾輪都要用接近 base input 的價格重新暖 cache。

**來源**：`README.md:121-124`

### Sesshu 的做法

用「便宜的本機小模型」取代「昂貴的 frontier model 全文摘要」：

1. Stop hook 在每輪之後，把 transcript **增量**（只送新增部分）丟給本機 LM。
2. 本機 LM 產出一份 12 段的 wiki 式摘要，存成 `seed.md` + 永久 wiki 頁。
3. transcript 超過門檻時，提示你可以 `/clear`。
4. 你 `/clear` 之後，SessionStart hook 把摘要 + 最近幾輪對話注入新 session。

**來源**：`README.md:62-86`、`src/sesshu/core.py:188-334`（`run_ingest`）、`src/sesshu/core.py:598-683`（`run_start`）

---

## 2. 名詞速查

| 名詞 | 意思 |
|---|---|
| **seed** | 暫存的壓縮摘要檔（`.sesshu/seed.md`），`/clear` 後被注入然後刪除 |
| **wiki page** | 永久保存的 session 摘要（`.sesshu/wiki/sessions/YYYY-MM-DD-HHMMSS-session.md`） |
| **ingest** | 把 transcript 新增部分送給 LM、產生／更新摘要的動作 |
| **cursor** | 記錄「已經 ingest 到第幾個字元」的檔案（`.sesshu/ingest/cursor.json`） |
| **HITL mode** | 預設模式：Stop hook + 你手動 `/clear` |
| **auto mode** | PreCompact/PreCompress hook，全自動、不提示使用者 |
| **structured events** | 不靠 LM、直接從 transcript 抽出的檔案編輯／git／bash／error 清單 |

---

## 3. 摘要長什麼樣：12 個固定段落

LM 被要求產出這 12 個 heading，缺一就重試（最多 2 次）：

```
## Goal                 ## Code changes
## Tasks                ## Git operations
## Plans                ## Errors & resolutions
## Decisions made       ## Blockers
## Rejected Approaches  ## External References
## Constraints          ## Current state
```

沒資訊的段落填 `_(not recorded)_`。

**來源**：`src/sesshu/structure.py:4-17`（`REQUIRED_SECTIONS`）、
`src/sesshu/structure.py:34-51`（`call_lm_with_retry`）、`README.md:70`

---

## 4. 快速導覽

```
你在這裡 → README.md（是什麼、值不值得裝）
              ↓
        architecture.md（怎麼設計的）
              ↓
   installation-and-usage.md（怎麼裝、怎麼用）
              ↓
        workflows.md（跑起來的時序）
              ↓
   code-map.md / source-index.md（怎麼改）
              ↓
      troubleshooting.md（壞掉怎麼辦）
```

三個最該先知道的設計約束（**來源已確認**）：

1. **所有 hook 出口一律 exit 0。** 任何失敗（LM 掛掉、transcript 壞掉、檔案寫不進去）
   都靜默返回 0，絕不阻擋宿主 runtime。
   來源：`CLAUDE.md`「Coding Constraints」、`AGENTS.md:161`、`src/sesshu/lm.py:52-72`（全 catch）
2. **runtime 只用標準函式庫**，沒有任何第三方相依。
   來源：`pyproject.toml:34-35`
3. **hook 絕不寫入、截斷或刪除 transcript `.jsonl` 檔。**
   來源：`CLAUDE.md`「Coding Constraints」、`AGENTS.md:163`

---

## 5. 適用情境

**【來源已確認】**（`README.md:225-244`）

✅ **適合**：

- 長時間、多輪的 coding session（重構、debug、大型 migration）
- 你願意在提示出現時手動 `/clear`（HITL mode 的效益完全綁在這件事上）
- 手邊有本機 LM（Ollama / LM Studio / vLLM），壓縮成本為 0
- 處理敏感／專有程式碼 → 本機 endpoint 讓 transcript 不出機器
- OSS 維護型工作：issue triage、bug fix、重構、測試修復、release 準備、文件更新
  （來源 `README.md:52-54` 明列此為目標場景）

❌ **不適合 / 省不到錢**：

- **短 session**（10 輪就結束，根本碰不到門檻）
- **output 主導的 session**（狂寫 code 而非狂讀 context，input 成長慢）
- **每一輪歷史都必須逐字保留的工作**——摘要若漏掉關鍵資訊，Claude 會反覆重問，
  來源文件說在對抗性 workload 上，42% 的節省可能被侵蝕到 20–30%
- **無人值守的批次執行**——HITL mode 的 `/clear` 需要人在迴圈裡
  （auto mode 可以無人值守，但走的是不同流程，見 [workflows.md](workflows.md)）

---

## 6. 成本效益：來源怎麼說

⚠️ **重要**：來源 `README.md:109` 自己標註了 pricing note——
**下列金額是示範性質，不是現價指引**，用的是單一參考價格模型與 prompt-cache 假設。
要引用前請用供應商當下的價格重算。

| 情境 | 每個長 session 估計成本 | 相對基準 |
|---|---|---|
| A — 不裝 Sesshu | $7.85 | — |
| B — 裝了且接受 `/clear` | $4.55 | **−42%** |
| C — 裝了但忽略 `/clear` | $7.85 | 0% |

**來源**：`README.md:196-200`

關鍵洞見（**來源已確認**，`README.md:184-192`）：
**忽略 Sesshu 沒有成本下降風險**。若你不 `/clear`，hook 只寫一個 seed 檔（無 API 成本）
再送一則約 50 token 的 `systemMessage`，其餘行為與沒裝一樣。

敏感度（`README.md:206-215`）：

| 每個 session 的 `/clear` 次數 | 估計降幅 |
|---|---|
| 從不 | 0% |
| 接近 auto-compact 時一次 | 15–20% |
| 中途兩次 | 40–45% |
| 三次（超長 session） | 50–55% |

超過三次報酬遞減，因為 cache 重新暖機的成本開始主導。

**訂閱制的影響**（`README.md:232-238`）：Pro/Max 月費是容量上限而非計量制，
Sesshu 不直接減少月費；效果是間接的——同樣配額能做更多事、
超額 burst 到 pay-as-you-go 時才是真的省錢、
以及有機會停在 Max 5x 而非 Max 20x（來源估年差 $1,200/開發者）。

---

## 7. 一個必須知道的成本副作用

**【來源已確認】**（`README.md:217-219`）

Sesshu 會**略微降低平均 cache hit rate**（約 95% → 88–92%），
因為每次 `/clear` 都讓 cache 失效、注入的摘要要幾輪才重新暖起來。
這是 Sesshu **唯一會增加成本**的機制，來源評估為每 session 幾分錢，
被輪次累積的節省蓋過。

---

## 8. 隱私邊界（務必先讀）

**【來源已確認】**（`README.md:559-579`、`AGENTS.md:233-235`）

- Stop hook 會把 **transcript 的一段切片**（最多 `SESSHU_MAX_INPUT` 字元）
  當作 user message 送到 `SESSHU_LM_URL` 的 `/chat/completions`。**除此之外不送其他資料。**
- 若 `SESSHU_LM_URL` 指向遠端服務（OpenRouter、OpenAI、雲端 vLLM…），
  那段切片——**可能含程式碼、檔案內容、對話上下文**——會經網路送到該供應商。
- **建議**：處理專有程式碼／機密資料時用本機 endpoint。預設值就是本機。
- `SESSHU_DEBUG=1` 的 debug log **不記錄** transcript 內容、產生的摘要或 API key，
  只記錄 timing、字數與檔案路徑。
- **視為敏感的衍生資料**：`.sesshu/seed.md`、`.sesshu/wiki/sessions/`、
  `.sesshu/wiki/search.db`（FTS5 索引裡存了 wiki 頁內容）、
  `.sesshu/related_state.json`、`.sesshu/related_notice.json`。

**供應鏈隔離**（`README.md:573-575`、`src/sesshu/config.py:161-170`）：
當 `SESSHU_DATA_DIR` 未設定時，`.env` 和 `prompts.json` **只從 `~/.sesshu/` 讀取，
永遠不從專案內的 `.sesshu/` 讀**。這阻止了惡意 repo 靠放一個 `.sesshu/.env`
就把 `SESSHU_LM_URL` 改掉來外洩 transcript。

**Hook 信任模型**（`README.md:567-571`）：
Sesshu hook 是註冊在 runtime `settings.json` 裡的 Python 腳本，
宿主 agent 每輪／每次 session 啟動都會以子行程執行它。
**安裝 Sesshu 等於給它跟其他任何 hook 一樣的信任層級。**
在意的話，安裝前先審 `hooks/` 與 `src/sesshu/`。

---

## 9. 一句話總結

> 用便宜的本機小模型，把「反覆重送的長歷史」換成「一份結構化摘要 + 最近幾輪」，
> 換取長 session 的 input 成本下降——效益完全取決於你願不願意按下 `/clear`。

---

**下一步** → [architecture.md](architecture.md)
