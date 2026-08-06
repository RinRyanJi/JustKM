# 安裝與使用

> 回到 [知識區入口](_index.md) ｜ 上一篇 [architecture.md](architecture.md) ｜ 下一篇 [workflows.md](workflows.md)

> ⚠️ 本文**不含任何真實秘密**。所有 API key 一律以佔位符表示。
> Ollama 的預設值 `ollama` 不是真實憑證，是 Ollama 慣例上被忽略的 dummy 值。

---

## 1. 前置需求（**來源已確認**，`README.md:246-251`）

- Python 3.9 或更新
- Claude Code / Codex / Gemini CLI / GitHub Copilot CLI 其中之一，且支援對應 hook 事件
- 一個 OpenAI 相容 endpoint：Ollama、LM Studio、vLLM、OpenRouter、OpenAI
- runtime 程式碼**不需要任何第三方 Python 套件**

---

## 2. 三種安裝途徑

| 途徑 | 指令 | 適用 | 來源 |
|---|---|---|---|
| **A. pip（推薦）** | `pip install sesshu` → `sesshu install` | 一般使用者，跨平台 | `README.md:255-269`、`src/sesshu/cli.py` |
| **B. 原始碼 + shell** | `bash ./install.sh` | Linux/macOS，開發 Sesshu 本身 | `README.md:329-337`、`install.sh` |
| **C. 原始碼 + PowerShell** | `.\install.ps1` | Windows，開發 Sesshu 本身 | `install.ps1`、`CHANGELOG.md`（Unreleased） |

### 2.1 途徑 A：pip 安裝（Windows / Linux / macOS 通用）

```bash
# 1. 安裝套件
pip install sesshu

# 2. 安裝 hook（預設 --runtime claude --scope user）
sesshu install --runtime claude --scope user

# 3. 產生設定檔範本
sesshu config init

# 4. 編輯 ~/.sesshu/.env，設定 SESSHU_LM_URL / SESSHU_LM_MODEL / SESSHU_LM_API_KEY

# 5. 重啟 runtime，讓新 hook 設定與 /sesshu-config 生效
```

用本機 Ollama 的話，`sesshu config init` 產生的預設值拉完模型就能直接用：

```bash
ollama pull qwen2.5:7b
```

**runtime 與 scope 選項**（`src/sesshu/cli.py:11-20, 302-318`）：

```bash
sesshu install --runtime claude|codex|gemini|copilot|both|all --scope user|repo
```

- `both` = claude + codex
- `all` = claude + codex + gemini + copilot
- 預設：`--runtime claude --scope user`

pip 安裝寫入的 hook 註冊指令長這樣（`cli.py:107-111`）：

```
SESSHU_DATA_DIR=<路徑> <目前的 python 執行檔> -m sesshu.hooks.claude_stop
```

用「目前的 python 執行檔」是刻意的——hook 會綁在你 `pip install sesshu` 的那個環境，
包含 virtualenv（`README.md:325`）。

> ### ⚠️【推論】Windows 上 pip 安裝的 hook 指令格式
>
> `_module_command()`（`cli.py:107-111`）產生的是
> `VAR=value command` 這種 **POSIX shell 前置環境變數**語法。
> 這在 `cmd.exe` / PowerShell 不成立。
>
> 相對地，`install.ps1` 的 `BuildCmdCommand` 明確產生
> `cmd /c "set ""SESSHU_DATA_DIR=..."" && set ""PYTHONPATH=..."" && python ""<script>"""`。
>
> **推論**：在 Windows 上，途徑 C（`install.ps1`）產生的指令格式才是為 Windows 設計的；
> 途徑 A 的指令格式是否能在 Windows runtime 下正確執行，**取決於該 runtime 如何 spawn
> hook 子行程**，來源文件未說明。這一點**未經實測驗證**。
> 若在 Windows 上用 pip 安裝後 hook 沒作用，請先檢查 `settings.json` 裡的 command 字串
> （見 [troubleshooting.md](troubleshooting.md) §2）。

### 2.2 途徑 B：Linux / macOS 原始碼安裝

```bash
git clone https://github.com/Nexplico-Ltd/Sesshu.git
cd Sesshu
bash ./install.sh --runtime claude --scope user
```

參數（`CLAUDE.md`「Commands」、`AGENTS.md:114`）：

```
./install.sh [--runtime claude|codex|gemini|copilot|both|all] [--scope user|repo] [--repo PATH]
```

行為：把 `hooks/` 的 wrapper 與 `src/sesshu` 套件**複製**進各 runtime 的 `hooks/` 目錄
（`install.sh:147-167`），再 patch 設定檔。需要 PATH 裡有 `python3`。

### 2.3 途徑 C：Windows PowerShell 原始碼安裝

```powershell
cd D:\path\to\Sesshu
.\install.ps1 -Runtime claude -Scope user
```

參數（`install.ps1:1-9`）：`-Runtime claude|codex|gemini|copilot|both|all`、
`-Scope user|repo`、`-Repo <路徑>`、（uninstall 另有 `-RemoveData`）

行為（`install.ps1`）：

1. 偵測 `python3` 或 `python`，都找不到就報錯退出
2. 資料目錄取 `$env:SESSHU_DATA_DIR`，否則 `%USERPROFILE%\.sesshu`
3. 建立 `<DataDir>\wiki\sessions`，複製 `data\prompts.json`（**已存在則跳過**）
4. 複製三個 wrapper（stop / start / precompact）到 `<ConfigDir>\hooks\`
5. 複製 `src\sesshu\*.py` 與 `src\sesshu\adapters\*.py` 到 `<ConfigDir>\hooks\sesshu\`
6. 安裝 `commands\sesshu-config.md`（**Copilot 跳過**），替換 `__SESSHU_ENV_PATH__`
7. 呼叫 `scripts\update_json_config.py` 更新 `settings.json` / `hooks.json`

Windows 各 runtime 設定目錄（`install.ps1`，user scope）：

| Runtime | 目錄 | 覆寫用環境變數 |
|---|---|---|
| Claude Code | `%USERPROFILE%\.claude` | `CLAUDE_DIR` |
| Codex | `%USERPROFILE%\.codex` | `CODEX_DIR`，其次 `CODEX_HOME` |
| Gemini CLI | `%USERPROFILE%\.gemini` | `GEMINI_DIR` |
| Copilot CLI | `%USERPROFILE%\.copilot` | `COPILOT_DIR` |
| Sesshu 資料 | `%USERPROFILE%\.sesshu` | `SESSHU_DATA_DIR` |

---

## 3. 安裝後的檔案落點

**user scope**（`README.md:317`、`cli.py:40-47`）：

```
~/.claude/settings.json     ← Stop / PreCompact / SessionStart hook 註冊
~/.claude/commands/sesshu-config.md
~/.codex/hooks.json         ← 注意：Codex 用 hooks.json，不是 settings.json
~/.gemini/settings.json     ← AfterAgent / PreCompress / SessionStart
~/.copilot/settings.json    ← 無 command 檔
~/.sesshu/                  ← 所有 Sesshu 資料
```

**repo scope**：把上面每個 `~/.X` 換成 `<repo>/.X`。

### `~/.sesshu/` 目錄結構（**來源已確認**，`README.md:472-485`、`ingest_queue.py:24-41`）

| 路徑 | 性質 | 內容 |
|---|---|---|
| `prompts.json` | 使用者可編輯 | LM prompt 模板（安裝時複製，**不覆寫既有檔**） |
| `.env` | 使用者可編輯 | 設定覆寫 |
| `seed.md` | 暫存 | 壓縮後的 seed，`/clear` 後刪除 |
| `shown.flag` | 暫存 | 去重旗標，`/clear` 後刪除 |
| `ingest/cursor.json` | 暫存 | `{session_id, chars_processed, last_ingest_ts}` |
| `ingest/queue.jsonl` | 暫存 | 背景 ingest 佇列 |
| `ingest/status.json` | 暫存 | 一次性背景 ingest 狀態 |
| `ingest.lock` | 暫存 | 背景 worker 鎖（**注意在 `data_dir` 根，不在 `ingest/` 下**） |
| `related_state.json` | 暫存 | 相關 session 偵測狀態（僅 `SESSHU_RELATED=1` 時寫） |
| `related_notice.json` | 暫存 | 一次性相關 session 通知 |
| `debug.jsonl` | 診斷 | 僅 `SESSHU_DEBUG=1` 時寫，1 MB 輪替 |
| `wiki/` | **永久** | wiki 頁、`index.md`、`log.md` |
| `wiki/search.db` | 衍生快取 | FTS5 索引，**刪掉安全** |

檔案權限：目錄 `0o700`、檔案 `0o600`（`io.py:85-127`、`cli.py:114-129`）。

---

## 4. 設定觀念

### 4.1 三個必須先懂的概念

**概念一：`SESSHU_DATA_DIR` 是特殊的。**
它只從 process environment 讀，**不從 `.env` 讀**，也不在 `DOTENV_KEYS` 白名單裡
（`config.py:54-73`）。把它寫進 `.sesshu/.env` **無效**。
理由：Sesshu 必須先知道要讀哪個 `.env`，才能讀 `.env`。
（來源：`README.md:487`、`AGENTS.md:200`）

**概念二：config 目錄 ≠ data 目錄。**
未設 `SESSHU_DATA_DIR` 時：`.env` / `prompts.json` **只從 `~/.sesshu/` 讀**，
而 seed / wiki 等 runtime 資料**會優先用當前工作目錄的 `.sesshu/`（若存在）**。
這是刻意的供應鏈隔離設計（`config.py:161-181`，見 [architecture.md](architecture.md) §8）。

**概念三：優先序。**（`README.md:364-369`）

```
process env SESSHU_*  >  config_dir/.env  >  process env CC_*（legacy）  >  預設值
```

⚠️ legacy `CC_*` 只從 process env 讀，**不吃 `.env`**（`config.py:145-158`）。

### 4.2 常用變數（**來源已確認**，`config.py:37-52`、`README.md:381-401`）

| 變數 | 預設 | 用途 |
|---|---|---|
| `SESSHU_LM_URL` | `http://localhost:11434/v1` | OpenAI 相容 endpoint base URL |
| `SESSHU_LM_MODEL` | `qwen2.5:7b` | 模型名稱 |
| `SESSHU_LM_API_KEY` | `ollama` | API key（Ollama 慣例的 dummy 值） |
| `SESSHU_THRESHOLD` | `120000` | 觸發提示的 transcript 字元數 |
| `SESSHU_MAX_INPUT` | `60000` | 送給 LM 的最大字元數 |
| `SESSHU_MAX_WORDS` | `600` | 摘要目標字數 |
| `SESSHU_TIMEOUT` | `90` | LM 請求逾時（秒） |
| `SESSHU_KEEP_TURNS` | `4` | 逐字附在 seed 後的最近輪次對 |
| `SESSHU_MODE` | `hitl` | `hitl` 或 `auto` |
| `SESSHU_INGEST_MIN_INTERVAL` | `300` | 兩次 LM ingest 的最短間隔（秒）；`0` 關閉 |
| `SESSHU_INGEST_MODE` | `sync` | `sync` 同步／`background` 背景 worker |
| `SESSHU_INGEST_WAIT_SECONDS` | `5` | SessionStart 等背景 worker 的秒數（clamp 到 0–30） |
| `SESSHU_INGEST_CHUNK_CHARS` | `20000` | 單次 ingest 處理的字元上限（最小 1） |
| `SESSHU_INGEST_MAX_RETRIES` | `3` | 背景佇列失敗記錄的重試上限 |
| `SESSHU_DEBUG` | *(空)* | `1` 開啟診斷 |
| `SESSHU_VERBOSE` | *(空)* | `1` 開啟額外提示與 restore 通知 |
| `SESSHU_RELATED` | *(空)* | `1` 開啟相關 session 偵測（opt-in） |
| `SESSHU_RELATED_THRESHOLD` | `0.3` | 相關性分數門檻 0.0–1.0 |
| `SESSHU_DATA_DIR` | `~/.sesshu` | **只從 process env 讀** |

隱藏變數：`SESSHU_INGEST_SYNC=1` 讓 `spawn_ingest_worker()` 改跑 inline
（`async_ingest.py:163, 175-177`）——來源文件未列在設定表中，**主要供測試用【推論】**。

`_bool_value()` 只接受 `"1"` 或 `"true"`（不分大小寫）為真（`config.py:106-107`）。

### 4.3 `.env` 格式（**來源已確認**，`config.py:117-142`）

- 每行 `KEY=value`
- `#` 開頭為註解行
- 支援行內註解（`# ...` 之後被截掉）—— **除非** 值被引號包住
- 支援 `export KEY=value` 前綴
- **只有白名單內的 18 個 `SESSHU_*` key 會被讀取**，其他一律忽略

範例（**佔位符，非真實憑證**）：

```dotenv
SESSHU_LM_URL=http://localhost:11434/v1
SESSHU_LM_MODEL=qwen2.5:7b
SESSHU_LM_API_KEY=<你的 API KEY 或 Ollama 的 ollama>
SESSHU_THRESHOLD=120000
SESSHU_MAX_INPUT=60000
SESSHU_KEEP_TURNS=4
SESSHU_TIMEOUT=90
SESSHU_INGEST_MIN_INTERVAL=300
SESSHU_INGEST_MODE=sync
# SESSHU_DEBUG=1
# SESSHU_VERBOSE=1
```

> 🔒 **絕對不要把填了真實 key 的 `.env` commit 進版控或貼進知識庫。**
> 檔案權限應為 `0o600`（`cli.py:144-147` 會嘗試設定，失敗則靜默忽略）。

### 4.4 `/sesshu-config` slash command

安裝於 Claude Code、Codex、Gemini CLI（**Copilot 沒有**，`cli.py:288`）：

```
/sesshu-config                    # 列出目前所有設定（分「常用」「進階」兩組）
/sesshu-config SESSHU_DEBUG=1     # 設 boolean
/sesshu-config SESSHU_DEBUG=      # 清除 boolean
/sesshu-config SESSHU_TIMEOUT=120 # 設整數
/sesshu-config SESSHU_MODE=auto   # 設 enum
```

它是一份 prompt 檔（`commands/sesshu-config.md`），內含完整驗證規則表。
一次只接受一個 `KEY=VALUE`，會驗證後寫回 `.env` 並顯示 before/after diff。
`SESSHU_DATA_DIR` **刻意不由此指令管理**（`commands/sesshu-config.md:11`）。

Copilot 使用者請直接編輯 `.env`（`README.md:445`）。

### 4.5 門檻建議（**來源已確認**，`README.md:501-506`）

| Context window | 建議 `SESSHU_THRESHOLD` |
|---:|---:|
| 200K tokens | `120000` – `180000` 字元 |
| 1M tokens | `350000` – `500000` 字元 |

驗證 hook 是否運作時先設很低（例如 `SESSHU_THRESHOLD=1000`），
通過真實 `/clear` smoke test 後再調高。

---

## 5. 日常使用流程（HITL mode）

```
1. 正常寫 code，Sesshu 在背景每輪增量 ingest（受 MIN_INTERVAL 節流）
        ↓
2. transcript 超過 SESSHU_THRESHOLD 時，出現一則 systemMessage
   例：sesshu: context compressed (145K chars). Run /clear to resume.
        ↓
3. 你決定：
   (a) 執行 /clear  → 新 session 自動收到摘要 + 最近 4 輪 + wiki hint，繼續工作
   (b) 忽略        → 什麼都不會壞，提示不會重複出現（shown.flag 擋住）
        ↓
4. 過往 session 摘要永久保存在 ~/.sesshu/wiki/sessions/，可隨時翻閱
```

**注入的內容包含什麼**（`core.py:662-663`、`wiki.py:86-110`）：

```
<seed_header（含時間戳）>
<12 段 LM 摘要>
## Recent turns (last 4)
**User:** ...（每則截斷至 800 字元）
**Assistant:** ...
<wiki_hint_header>
Recent session pages available for detailed lookup:
  - <路徑1> … 最多 5 筆
Full index: ~/.sesshu/wiki/index.md
Full log:   ~/.sesshu/wiki/log.md
<wiki_hint_footer>
```

---

## 6. 自訂 prompt

所有 LM prompt 與訊息模板都在 `<config_dir>/prompts.json`。
安裝時從 `data/prompts.json` 複製，**後續安裝不覆寫**，你的編輯會保留
（`cli.py:122-129`）。

六個模板 key（**來源已確認**，實際讀自 `data/prompts.json`）：

| Key | 用途 | 可用變數 |
|---|---|---|
| `stop.create` | 建立新 wiki 頁 | `{excerpt}` `{max_words}` `{structured_events}` |
| `stop.update` | 更新既有 wiki 頁 | `{existing}` `{excerpt}` `{max_words}` `{structured_events}` |
| `stop.system_message` | 給使用者看的提示 | `{size}` |
| `start.seed_header` | 注入內容的表頭 | `{timestamp}` |
| `start.wiki_hint_header` | wiki hint 區塊表頭 | — |
| `start.wiki_hint_footer` | wiki hint 區塊表尾 | — |

⚠️ **升級注意**（`README.md:451`、`AGENTS.md:208`）：
既有的自訂 `prompts.json` 必須自行在 `stop.create` 和 `stop.update` 加上含
`{structured_events}` 的 `STRUCTURED EVENTS:` 區塊。沒加的話 Sesshu 照跑，
但預先抽出的檔案編輯 / git / bash / error 不會被注入 prompt。

`prompts.json` 缺失或格式錯誤時，兩個 hook 都靜默 exit 0，**不做任何壓縮**
（`core.py:210-213`、`core.py:642-645`）。還原預設：

```bash
cp data/prompts.json ~/.sesshu/prompts.json
```

---

## 7. 移除

```bash
# pip 安裝
sesshu uninstall --scope user
sesshu uninstall --runtime all --scope user --remove-data   # 連資料一起刪

# Linux/macOS 原始碼安裝
bash ./uninstall.sh --scope user

# Windows 原始碼安裝
.\uninstall.ps1 -Runtime claude -Scope user
```

預設**保留** `.sesshu/` 資料（seed、wiki、`prompts.json`），
只移除 hook 註冊與 `/sesshu-config` 指令檔（`README.md:348`、`cli.py:280-299`）。
加 `--remove-data` / `-RemoveData` 才會 `shutil.rmtree()` 整個資料目錄。

⚠️ 用哪個 scope 裝就用哪個 scope 移除。

---

## 8. 開發者常用指令（**來源已確認**）

```bash
pytest -q                                        # 跑全部測試
pytest tests/test_stop_hook.py -q                # 跑單一檔
python3 -m py_compile src/sesshu/*.py tests/*.py # 只做語法檢查（pytest 不可用時）

# 手動執行 hook（hook JSON 走 stdin）
PYTHONPATH="$PWD/src" python3 hooks/sesshu_claude_stop.py
SESSHU_DEBUG=1 PYTHONPATH="$PWD/src" python3 hooks/sesshu_claude_stop.py

# 測 LM 延遲
python3 scripts/measure_latency.py --runs 3
python3 scripts/measure_latency.py --url "$SESSHU_LM_URL" --models qwen2.5:7b llama3.2:3b --runs 3

# Benchmark
python3 scripts/benchmark_suite.py --dry-run
python3 scripts/benchmark_suite.py --aggregate benchmarks/results/results.jsonl
```

來源：`CLAUDE.md`「Commands」、`AGENTS.md:102-118`、`README.md:513-541`

`pyproject.toml:61-64` 設了 `pythonpath = ["src", "."]`，
所以**跑 pytest 不需要手動設 `PYTHONPATH`**。

> **【來源已確認】測試現況說明**：本知識區**未執行** `pytest`。
> 來源專案有 30 個 `tests/test_*.py`，但本次整理沒有跑過測試，
> 因此不宣稱任何通過率。`README.md:551` 也提醒：若 pytest 不可用，
> 跑 `py_compile` 並明確說明測試未執行。

---

**下一步** → [workflows.md](workflows.md)
