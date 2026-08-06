# 原始碼索引

> 回到 [知識區入口](_index.md) ｜ 目錄總覽 [code-map.md](code-map.md) ｜ 問題排查 [troubleshooting.md](troubleshooting.md)

路徑皆相對於來源專案根目錄 `D:/AiWorkSpace/Sesshu-main`，且皆為實際存在的檔案。
本頁只談「讀什麼、為什麼」；每個檔案的行數與細節見 [code-map.md](code-map.md)。

---

## 建議閱讀順序

### 階段 1 — 先弄懂契約（約 300 行）

| # | 檔案 | 讀它是為了 |
|---|---|---|
| 1 | `src/sesshu/config.py` | 所有 `SESSHU_*` 變數的**唯一真相來源**：`Config` frozen dataclass、預設值、`DOTENV_KEYS` 白名單、config 目錄與 data 目錄的雙軌選擇邏輯 |
| 2 | `src/sesshu/runtime.py` | `normalize_runtime()` / `normalize_scope()` / `data_dir_for_scope()`，四種 runtime 與 user/repo scope 的名稱正規化 |
| 3 | `src/sesshu/io.py` | hook 的 I/O 契約：`read_hook_input()` 讀 stdin JSON、`emit_json()` 寫 stdout、`debug_log()`、私有目錄與原子寫入 |

> 先讀這三個，後面所有模組的參數才不會看得一頭霧水。

### 階段 2 — 主幹流程（最重要）

| # | 檔案 | 讀它是為了 |
|---|---|---|
| 4 | `src/sesshu/core.py` | **心臟**。`run_ingest()` / `run_stop()` / `run_start()` / `run_precompact()` 四個 runtime-neutral 編排函式，加上 `HookAdapter` Protocol。想理解 Sesshu 只需要讀懂這一個檔 |
| 5 | `src/sesshu/adapters/claude_code.py` | 基底 adapter：怎麼從 hook payload 取出 transcript 路徑與 source |
| 6 | `src/sesshu/adapters/copilot.py` | 差異最大的 adapter（pseudo-transcript、source 正規化、decision 輸出），對照它最能看出 Protocol 的抽象邊界 |

> `adapters/codex.py` 與 `adapters/gemini.py` 覆寫的東西很少，先跳過。

### 階段 3 — 支線能力（依需求挑）

| 主題 | 檔案 |
|---|---|
| LM 呼叫與結構驗證 | `src/sesshu/lm.py` → `src/sesshu/structure.py` |
| Prompt 模板載入 | `src/sesshu/prompts.py` + `data/prompts.json` |
| Transcript 解析 | `src/sesshu/transcript.py` → `src/sesshu/events.py` → `src/sesshu/fallback.py` |
| Wiki 落地與檢索 | `src/sesshu/wiki.py` → `src/sesshu/search.py` |
| 相關 session 偵測 | `src/sesshu/related.py` |
| 背景 ingest | `src/sesshu/async_ingest.py` → `src/sesshu/ingest_queue.py` → `src/sesshu/ingest_worker.py` |
| 安裝流程 | `src/sesshu/cli.py` |

### 階段 4 — 對照測試理解行為

讀完某個模組後，直接看 `tests/` 中同名的 `test_*.py`（例如 `src/sesshu/search.py` → `tests/test_search.py`）。
`tests/conftest.py` 的 `fake_lm_server` fixture 是理解 LM 呼叫路徑最快的入口。

---

## `src/` — runtime 程式碼

### 編排層

| 檔案 | 實際用途 |
|---|---|
| `src/sesshu/core.py` | 四個 runtime-neutral 編排函式，全部 hook 行為都在這裡 |
| `src/sesshu/stop_hook.py` | 薄 shim：Claude adapter + `core.run_stop()` |
| `src/sesshu/start_hook.py` | 薄 shim：Claude adapter + `core.run_start()` |
| `src/sesshu/precompact_hook.py` | 薄 shim：Claude adapter + `core.run_precompact()` |

### 設定與 I/O

| 檔案 | 實際用途 |
|---|---|
| `src/sesshu/config.py` | 載入 `SESSHU_*` 環境變數與受信任 `.env`，回傳 `Config` |
| `src/sesshu/io.py` | stdin/stdout hook 協定、debug log 輪替、私有目錄與原子寫入 |
| `src/sesshu/prompts.py` | 讀 `prompts.json`；失敗回 `{}`，`get_prompt()` 做模板展開 |
| `src/sesshu/runtime.py` | runtime / scope 名稱正規化與資料目錄推導 |

### LM 互動

| 檔案 | 實際用途 |
|---|---|
| `src/sesshu/lm.py` | 只用 `urllib.request` 打 OpenAI 相容 `/chat/completions`；任何失敗回 `""` |
| `src/sesshu/structure.py` | 12 個必要 heading 的驗證與最多 2 次的修正重試 |

### Transcript 與事件

| 檔案 | 實際用途 |
|---|---|
| `src/sesshu/transcript.py` | 純讀 `.jsonl` transcript、抽取最近幾輪對話 |
| `src/sesshu/events.py` | 從 diff 抽出 file edits / git ops / errors / bash commands，格式化後注入 LM prompt |
| `src/sesshu/fallback.py` | 無 LM 時產生 12 段齊全的 wiki 頁；並 re-export `events.py` 的符號以維持相容 |

### 持久化與檢索

| 檔案 | 實際用途 |
|---|---|
| `src/sesshu/wiki.py` | 組 seed 內容、寫 wiki 頁與 `index.md` / `log.md`、挑選相關頁面、組 wiki hint |
| `src/sesshu/search.py` | 關鍵字抽取、SQLite FTS5 索引維護與查詢（無 FTS5 時降級為 grep） |
| `src/sesshu/related.py` | 相關 session 評分與一次性通知決策（`SESSHU_RELATED=1` 才啟用） |

### 背景 ingest

| 檔案 | 實際用途 |
|---|---|
| `src/sesshu/async_ingest.py` | 鎖檔管理與背景 worker 生成；含 Windows 專用的 PID 存活檢查 |
| `src/sesshu/ingest_queue.py` | 佇列記錄結構與 `ingest/` 下各檔案的路徑輔助函式 |
| `src/sesshu/ingest_worker.py` | 子行程進入點：處理佇列、呼叫 ingest、釋放鎖 |

### 子套件

| 路徑 | 實際用途 |
|---|---|
| `src/sesshu/adapters/` | `claude_code.py`（基底）、`codex.py`、`gemini.py`、`copilot.py` — 吸收四種 runtime 的 payload 差異 |
| `src/sesshu/hooks/` | pip 安裝用的 module 進入點，供 `python -m sesshu.hooks.<runtime>_<event>` 呼叫 |
| `src/sesshu/assets/` | 打包進 wheel 的資源：`env.example`、`commands/sesshu-config.md`、`data/prompts.json` |
| `src/sesshu/cli.py` | `sesshu install` / `uninstall` / `config init` 的實作 |

---

## `hooks/` — 原始碼安裝用的 wrapper

12 個檔案，命名規則為 `sesshu_<runtime>_<event>.py`，runtime 為
`claude` / `codex` / `gemini` / `copilot`，event 為 `stop` / `start` / `precompact`。

每個檔只有 6 行，例如 `hooks/sesshu_claude_stop.py` 只做一件事：
`from sesshu.stop_hook import main` 然後 `raise SystemExit(main())`。

**用途區分**：`hooks/` 給 `install.sh` / `install.ps1`（原始碼安裝，用路徑呼叫）；
`src/sesshu/hooks/` 給 `sesshu install`（pip 安裝，用 `python -m` 呼叫）。
兩套並存不是重複，是兩種安裝方式的進入點。

---

## `commands/`

| 檔案 | 實際用途 |
|---|---|
| `commands/sesshu-config.md` | `/sesshu-config` slash command 的 prompt 檔：變數型別表、List Mode 顯示格式、Set Mode 驗證規則。安裝時把 `__SESSHU_ENV_PATH__` 佔位符換成實際 `.env` 路徑 |

同一份內容也存在 `src/sesshu/assets/commands/sesshu-config.md` 供 pip 打包使用。
Copilot CLI 沒有相容的 command 目錄，因此不安裝此檔（見 [troubleshooting.md](troubleshooting.md) §9）。

---

## `scripts/` — 獨立 CLI 工具

| 檔案 | 實際用途 |
|---|---|
| `scripts/__init__.py` | 讓 `scripts/` 成為正式 package，測試才能 `from scripts import benchmark_suite` |
| `scripts/benchmark_suite.py` | 產生 benchmark 執行計畫（`--dry-run`）、彙總 JSONL 結果（`--aggregate`）、評估 H1–H4 假說 |
| `scripts/measure_latency.py` | 量測 OpenAI 相容 endpoint 延遲，用來挑選夠快又填得滿 12 段的模型 |
| `scripts/update_json_config.py` | 被 `install.ps1` 呼叫，安全地增修 `settings.json` / `hooks.json`，不動其他 hook |

搭配資料在 `benchmarks/`：`benchmarks/README.md`（欄位定義）、
`benchmarks/tasks/`（4 份任務 prompt）、`benchmarks/results/`（輸出 JSONL）。

---

## `tests/` — pytest 測試

| 檔案 | 實際用途 |
|---|---|
| `tests/conftest.py` | 提供 `run_hook()` helper 與 `fake_lm_server` fixture（隨機 port 的 `ThreadingHTTPServer`）；任何走 LM 路徑的測試都用它 |

其餘 30 個 `test_*.py` 大致一個模組一檔，完整清單見 [code-map.md](code-map.md) §6。
挑幾個代表性的：

| 檔案 | 覆蓋什麼 |
|---|---|
| `tests/test_core.py` | 四個編排函式的主流程與各種退出路徑 |
| `tests/test_adapters.py` | Claude / Codex adapter 的 payload 解析 |
| `tests/test_copilot_adapter.py` | Copilot 的 pseudo-transcript 與 decision 輸出 |
| `tests/test_search.py` | FTS5 索引、grep 降級、三向對帳 |
| `tests/test_async_ingest.py` | 鎖檔取得、stale 判定、worker 生成 |
| `tests/test_cli_install.py` | `sesshu install` 的檔案落地與設定合併 |

測試慣例：檔案系統行為一律用 `tmp_path`；**不得改動真實的 Claude Code / Codex / Gemini CLI
設定或資料檔**。`pyproject.toml` 已設 `pythonpath = ["src", "."]`，跑 pytest 不需手動設 `PYTHONPATH`。

---

## 其他值得一讀的非程式碼檔

| 檔案 | 實際用途 |
|---|---|
| `README.md` | 動機、成本模型、安裝、完整設定表、安全性說明 |
| `CLAUDE.md` / `AGENTS.md` | 給 coding agent 的專案規範（注意其架構樹**不完整**，見 [code-map.md](code-map.md) §2.7） |
| `docs/smoke-test.md` | 真實 runtime 生命週期檢查清單（Claude Code 與 Codex 兩節） |
| `docs/sesshu-architecture-and-usage-guide.html` | 單一 HTML 長文，涵蓋架構到效能調校的完整章節 |
| `pyproject.toml` | 打包設定、pytest 設定、CLI entry point、wheel 要包的 assets |
| `.env.example` | `.env` 範本，**只用來確認變數名稱**；本知識區未複製任何值 |

---

**相關** → [troubleshooting.md](troubleshooting.md)
