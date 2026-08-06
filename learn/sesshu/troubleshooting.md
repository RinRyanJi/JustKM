# 疑難排解

> 回到 [知識區入口](_index.md) ｜ 上一篇 [code-map.md](code-map.md) ｜ 相關 [source-index.md](source-index.md)

路徑皆相對於來源專案根目錄 `D:/AiWorkSpace/Sesshu-main`。
本頁每項都可由來源文件或程式碼確認，並附出處。

> ⚠️ 以下所有檢查命令**只讀取檔名與檔案是否存在**，不印出 `.env` 內容。
> 若要確認設定值，請用 `/sesshu-config`（不支援 Copilot CLI），不要 `cat .env`。

---

## 1. 完全沒反應：hook 沒被註冊，或 runtime 沒重啟

**現象**：對話很長了，卻沒有任何 Sesshu 訊息，`~/.sesshu/` 也是空的。

**檢查**：
```bash
ls ~/.claude/hooks/sesshu_claude_stop.py    # 原始碼安裝
ls ~/.sesshu/
```

**預期**：`install.sh` 會把 wrapper 寫進 `~/.claude/hooks/`，並在 `~/.claude/settings.json`
註冊 Stop 與 SessionStart（`docs/smoke-test.md:17`）。

**限制**：**改完 hook 設定或環境變數後必須重啟 runtime**
（`README.md:327`、`docs/smoke-test.md:43`）。Codex 另需在 `/hooks` 內信任該 hook 層
（`docs/smoke-test.md:95`）。

---

## 2. `prompts.json` 遺失或格式錯誤 → 兩個 hook 都靜默退出

**現象**：完全無任何輸出、無錯誤訊息、也不產生 `seed.md`。

**檢查**：
```bash
ls ~/.sesshu/prompts.json
python3 -c "import json,pathlib;json.loads(pathlib.Path.home().joinpath('.sesshu/prompts.json').read_text())"
```

**預期**：`prompts.json` 缺失或壞掉時，兩個 hook 都以 exit code 0 靜默結束、不做壓縮
（`README.md:462`）。`load_prompts()` 任何失敗都回 `{}`，呼叫端就 `return (0, "")`
（`src/sesshu/prompts.py`、`CLAUDE.md`「Module Responsibilities」）。

**還原**：
```bash
cp data/prompts.json ~/.sesshu/prompts.json
```

**限制**：這是**刻意設計**——所有 hook 退出路徑都必須回 exit 0，否則會擋住 Claude Code
（`CLAUDE.md`「Coding Constraints」）。因此「靜默」不代表沒問題，要開 `SESSHU_DEBUG=1` 才看得到原因。

---

## 3. 舊的自訂 `prompts.json` 少了 `{structured_events}`

**現象**：功能正常，但摘要品質變差——檔案編輯、git 操作、錯誤都沒被抓進去。

**檢查**：
```bash
grep -c "{structured_events}" ~/.sesshu/prompts.json    # 應為 2
```

**預期**：`stop.create` 與 `stop.update` 兩處都要有 `STRUCTURED EVENTS:` 區塊含
`{structured_events}` 佔位符。少了它 **Sesshu 仍會跑**，只是預先抽取的事件不會注入 LM prompt
（`README.md:451`）。

**限制**：安裝時**不會覆寫**既有 `prompts.json`（為了保留使用者編輯，`README.md:449`），
所以升級後這個問題不會自動修好。

---

## 4. LM 連不上 / 逾時 → 降級成 LM-free fallback

**現象**：wiki 頁面產出了，但內容都是事件清單而非敘述式摘要。

**檢查**：
```bash
python3 scripts/measure_latency.py --runs 3
ls ~/.sesshu/wiki/sessions/
```

**預期**：`call_lm()` 遇到任何例外都回 `""`，讓呼叫端乾淨地 exit 0
（`CLAUDE.md`「Module Responsibilities」）。此時 `build_fallback_page()` 會產生
**12 段齊全但無 LM** 的 wiki 頁（`src/sesshu/fallback.py`、`CLAUDE.md` TODO 第 1 項已完成）。

**限制**：預設 `SESSHU_TIMEOUT=90` 秒（`README.md:390`）；LM 回應超過 1 MiB 也視為失敗
（`src/sesshu/lm.py`，見 [code-map.md](code-map.md) §2.3）。
另外若模型太小，12 個 section 可能填不滿——`call_lm_with_retry()` 最多重試 2 次
（`src/sesshu/structure.py`）。

---

## 5. 把 `SESSHU_DATA_DIR` 寫進 `.env` → **會被忽略**

**現象**：明明在 `.sesshu/.env` 設了 `SESSHU_DATA_DIR`，卻完全沒生效。

**預期**：`SESSHU_DATA_DIR` **只從 process 環境讀取**，因為 Sesshu 必須先知道要讀哪個 `.env`
（`README.md:487`）。程式面也印證：`DOTENV_KEYS` 白名單共 18 個 key，
**不含 `SESSHU_DATA_DIR`**（`src/sesshu/config.py:54-73`）。

**正確做法**：在 shell 中 `export`，或依賴安裝好的 hook 命令（安裝腳本會明確帶入）。

**限制（且是安全設計）**：`SESSHU_DATA_DIR` 未設定時，`.env` 與 `prompts.json`
**只從 `~/.sesshu/` 讀**，絕不從專案目錄的 `.sesshu/` 讀。
這是為了避免惡意 repo 靠放一個 `.sesshu/.env` 就竄改 `SESSHU_LM_URL` 外洩 transcript
（`README.md:575`「Config and supply-chain isolation」）。

---

## 6. `/clear` 後沒有注入 → source 或 mode 不對

**現象**：`/clear` 之後新 session 一片空白，`seed.md` 卻還留著。

**預期（HITL 模式）**：SessionStart 只在 `source == "clear"` 時動作，
其餘 source 直接退出（`src/sesshu/core.py:609-611`）：

```python
accepted = {"clear"} if config.mode == "hitl" else {"compact", "clear"}
```

也就是說**一般開新 session、resume 都不會注入**——這是預期行為，不是 bug。
auto 模式才會額外接受 `compact`。

**檢查 source**：照 `docs/smoke-test.md:70-79` 暫時用 `tee` 把 hook 輸入導到檔案，
確認含 `"source":"clear"`，測完記得還原。

**mode 交叉限制**（`src/sesshu/core.py:198, 356, 488`）：
| mode | `run_ingest` / `run_stop` | `run_precompact` |
|---|---|---|
| `hitl`（預設） | 執行 | 直接退出 |
| `auto` | 直接退出 | 執行 |

設成 `auto` 卻期待看到 `/clear` 提示，永遠不會出現（`README.md:90`）。

---

## 7. 一直沒有更新：`SESSHU_INGEST_MIN_INTERVAL` 時間閘

**現象**：連續對話很多輪，`seed.md` 的時間戳卻沒動。

**預期**：預設 `300` 秒內只會做一次 LM ingest（`README.md:393`）。
`cursor.json` 記錄 `{session_id, chars_processed, last_ingest_ts}`
（`README.md:478`）。除錯時設 `SESSHU_INGEST_MIN_INTERVAL=0` 可關掉這道閘。

**檢查**：
```bash
ls ~/.sesshu/ingest/
SESSHU_DEBUG=1 PYTHONPATH="$PWD/src" python3 hooks/sesshu_claude_stop.py < /dev/null
```

**限制**：`SESSHU_DEBUG=1` 的 `debug.jsonl` **只有時間、token 數、路徑，不含 transcript 內容、
摘要或 API key**（`README.md:579`），所以它幫不上「摘要內容怪怪的」這類問題。

---

## 8. 提示只出現一次；門檻沒到就完全沒聲音

**現象**：看到一次 `/clear` 建議後就再也沒有了；或是覺得對話夠長卻沒提示。

**預期**：
- 低於 `SESSHU_THRESHOLD`（預設 `120000` 字元）→ **靜默退出**，但 seed 仍保留給下次 `/clear`（`README.md:77`）。
- 超過門檻且 seed 存在 → 發出 `systemMessage` 並寫 `shown.flag` 抑制重複（`README.md:78`）。
- `shown.flag` 與 `seed.md` 在 `/clear` 後才被刪除（`README.md:86`）。

**檢查**：
```bash
ls ~/.sesshu/seed.md ~/.sesshu/shown.flag
```

**除錯建議**：先設 `SESSHU_THRESHOLD=1000` 驗證整條路徑通了，再調回實際值
（`README.md:506`、`docs/smoke-test.md:24`）。設 `SESSHU_VERBOSE=1` 可讓「未達門檻」
與「已提示過」這兩種退出也印出訊息（`README.md:392`）。

**限制**：`systemMessage` 的文字只能是「事實陳述 + 簡短建議」，
不可含 next-steps 或任務指示語——那是已知的無限迴圈觸發條件（`CLAUDE.md`「Coding Constraints」）。

---

## 9. Copilot CLI 沒有 `/sesshu-config`

**現象**：在 Copilot CLI 打 `/sesshu-config` 沒有反應。

**預期**：Copilot CLI 沒有相容的 command 目錄，所以安裝時**不會**放入 `/sesshu-config`；
Copilot 使用者需直接編輯 `.env`（`README.md:445`）。
Claude Code / Codex / Gemini CLI 三者才有（`README.md:433`）。

---

## 已知限制彙整

| 限制 | 出處 |
|---|---|
| 短 session（如 10 輪）永遠不觸發 | `README.md:227` |
| output 為主的 session 省不到錢 | `README.md:228` |
| 每輪都需要完整前文的工作，摘要會反而增加輪數 | `README.md:229` |
| 無人值守的批次執行不適用（HITL 需要人決定 `/clear`） | `README.md:230` |
| `/clear` 會使 prompt cache 失效，平均命中率約 95% → 88–92% | `README.md:219` |
| 兩個 hook 都不得寫入、截斷或刪除 `.jsonl` transcript | `CLAUDE.md`「Coding Constraints」 |
| `~/.claude/sesshu` 舊資料**不會**自動搬遷 | `README.md:489` |
| `wiki/search.db` 為自動建立，可安全刪除（會重建） | `README.md:485` |

---

**相關** → [source-index.md](source-index.md)
