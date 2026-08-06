# 程式碼地圖

> 回到 [知識區入口](_index.md) ｜ 上一篇 [workflows.md](workflows.md) ｜ 下一篇 [troubleshooting.md](troubleshooting.md)

以下路徑皆相對於來源專案根目錄 `D:/AiWorkSpace/Sesshu-main`。

---

## 1. 頂層目錄

| 路徑 | 用途 |
|---|---|
| `src/sesshu/` | 所有 runtime 程式碼（20 個模組 + 3 個子套件） |
| `hooks/` | 12 個 hook wrapper（原始碼安裝時複製到 runtime 的 `hooks/`） |
| `commands/` | `/sesshu-config` slash command 的 prompt 檔 |
| `scripts/` | 3 個 CLI 工具（明確的 Python package，含 `__init__.py`） |
| `data/` | `prompts.json` 預設模板（安裝時複製到資料目錄） |
| `tests/` | pytest 測試（`conftest.py` + 30 個 `test_*.py`） |
| `benchmarks/` | `tasks/` 4 份任務 prompt、`results/` 輸出 JSONL |
| `docs/` | `smoke-test.md`、`sesshu-architecture-and-usage-guide.html` |
| `install.sh` / `uninstall.sh` | Linux/macOS 安裝腳本 |
| `install.ps1` / `uninstall.ps1` | Windows PowerShell 安裝腳本 |
| `.env.example` | `.env` 範本（只有變數名，**本知識區未複製任何值**） |
| `pyproject.toml` | 打包設定、pytest 設定、CLI entry point |

---

## 2. `src/sesshu/` — 核心模組

### 2.1 編排層

| 檔案 | 行數 | 職責 |
|---|---:|---|
| `core.py` | 684 | **心臟**。`run_ingest()` `run_stop()` `run_start()` `run_precompact()` 四個 runtime-neutral 編排函式，加上 `HookAdapter` Protocol、`StopRequest` / `StartRequest` dataclass |
| `stop_hook.py` | 16 | 薄 shim：Claude adapter + `core.run_stop()` |
| `start_hook.py` | 16 | 薄 shim：Claude adapter + `core.run_start()` |
| `precompact_hook.py` | 16 | 薄 shim：Claude adapter + `core.run_precompact()` |

### 2.2 設定與 I/O

| 檔案 | 行數 | 重點 |
|---|---:|---|
| `config.py` | 276 | `Config` frozen dataclass（22 欄位）、18 個預設常數、`DOTENV_KEYS` 白名單、`_config_dir()` / `_default_data_dir()` 雙軌目錄選擇、`load_config()` |
| `io.py` | 163 | `read_hook_input()` `emit_json()` `debug_log()`（1 MB 輪替）、`ensure_private_dir()`（0o700）、`write_private_text()`（**原子寫入**，見 [architecture.md](architecture.md) §12） |
| `prompts.py` | 27 | `load_prompts()`（任何失敗回 `{}`）、`get_prompt()`（走訪 dict + `str.format_map()`） |
| `runtime.py` | 33 | `RuntimeName` Enum、`normalize_runtime()` `normalize_scope()` `data_dir_for_scope()` |

### 2.3 LM 互動

| 檔案 | 行數 | 重點 |
|---|---:|---|
| `lm.py` | 72 | `call_lm()`：只用 `urllib.request` 打 `/chat/completions`，`temperature=0.3`、`max_tokens=1200`、`stream=False`。回應以 8192 bytes 分塊讀，超過 `MAX_LM_RESPONSE_BYTES`(1 MiB) 視為失敗。任何例外都回 `""` |
| `structure.py` | 51 | `REQUIRED_SECTIONS`（12 個 heading）、`validate_sections()`、`build_retry_prompt()`、`call_lm_with_retry(max_retries=2)` |

### 2.4 Transcript 與事件

| 檔案 | 行數 | 重點 |
|---|---:|---|
| `transcript.py` | 58 | `read_transcript()`（純讀）、`extract_recent_turns()`（每則截 800 字元；同時支援 `type:"message"` 與 `type:"user"/"assistant"` + 巢狀 `message` 兩種 JSONL 格式） |
| `events.py` | 121 | `EventSnapshot` dataclass（`file_edits` / `git_ops` / `errors` / `bash_commands`）、`extract_events()`（遞迴走訪 `content` 與 `message`）、`format_structured_events()` |
| `fallback.py` | 24 | `build_fallback_page()`：無 LM 時產生 12 段齊全的 wiki 頁；並 re-export `EventSnapshot` / `extract_events` 以維持向後相容 |

### 2.5 持久化與檢索

| 檔案 | 行數 | 重點 |
|---|---:|---|
| `wiki.py` | 110 | `build_seed_content()`、`write_wiki_page()`（時間戳檔名 + 撞名加後綴 + 更新 `index.md` / `log.md`，log 1 MB 輪替）、`select_wiki_pages()`、`build_wiki_hint()` |
| `search.py` | 227 | `extract_keywords()`（stopword 過濾 + 頻率 top-20）、`update_search_index()`、`search_wiki()`（FTS5 → grep 降級）、`sync_missing_pages()`（新增/修改/刪除三向對帳）、`_grep_search()` |
| `related.py` | 104 | `RelatedHit`、`score_related()`、`find_related()`、`load_related_state()` / `write_related_state()`、`decide_notifications()`（`EPSILON = 0.1`） |

### 2.6 背景 ingest

| 檔案 | 行數 | 重點 |
|---|---:|---|
| `async_ingest.py` | 338 | `spawn_ingest_worker()`（原子建鎖 + 重試 3 次）、`is_lock_stale()`、`_is_pid_alive()`（**Windows 用 `OpenProcess` 而非 `os.kill`**）、`_run_inline()` |
| `ingest_queue.py` | 88 | `QueueRecord` frozen dataclass、路徑輔助（`ingest_dir` / `queue_path` / `cursor_path` / `status_path` / `lock_path`）、`read_queue()` `append_queue_record()` `compact_queue()` |
| `ingest_worker.py` | 150 | 子行程進入點：`_queue_payloads()` `_update_queue()` `_release_lock()`（token 比對）`main()` |

### 2.7 子套件

```
src/sesshu/adapters/       ← runtime 差異
    claude_code.py         基底 adapter
    codex.py               覆寫 parse_stop（多讀 last_assistant_message，目前保留未用）
    gemini.py              覆寫 build_start_output（payload 無 hookEventName）
    copilot.py             pseudo-transcript + source 正規化 + decision 輸出

src/sesshu/hooks/          ← pip 安裝的 module 進入點（python -m sesshu.hooks.X）
    claude_stop.py / claude_start.py / claude_precompact.py
    codex_*.py / gemini_*.py / copilot_*.py      （共 12 檔，每檔約 4 行）

src/sesshu/assets/         ← 打包進 wheel 的資源（pyproject.toml:56）
    env.example
    commands/sesshu-config.md
    data/prompts.json

src/sesshu/cli.py          ← sesshu install / uninstall / config init（334 行）
```

> **【文件不一致】**：來源的 `CLAUDE.md`「Architecture」樹狀圖**沒有列出**
> `cli.py`、`related.py`、`ingest_queue.py`、`assets/`、`hooks/` 子套件。
> `AGENTS.md:42-96` 的樹也一樣不完整。實際檔案請以本文為準。

---

## 3. `hooks/` — 12 個 wrapper

```
sesshu_claude_stop.py    sesshu_claude_start.py    sesshu_claude_precompact.py
sesshu_codex_stop.py     sesshu_codex_start.py     sesshu_codex_precompact.py
sesshu_gemini_stop.py    sesshu_gemini_start.py    sesshu_gemini_precompact.py
sesshu_copilot_stop.py   sesshu_copilot_start.py   sesshu_copilot_precompact.py
```

每個檔只有 6 行，例如 `hooks/sesshu_claude_stop.py`：

```python
#!/usr/bin/env python3
"""Explicit Claude Code wrapper for the Sesshu Stop hook."""
from sesshu.stop_hook import main

if __name__ == "__main__":
    raise SystemExit(main())
```

**兩套進入點的差別**（**來源已確認**）：

| | `hooks/*.py` | `src/sesshu/hooks/*.py` |
|---|---|---|
| 誰用 | `install.sh` / `install.ps1`（原始碼安裝） | `sesshu install`（pip 安裝） |
| 呼叫方式 | `python <path>/sesshu_claude_stop.py` | `python -m sesshu.hooks.claude_stop` |
| 相依取得 | 安裝腳本把 `src/sesshu` **複製**進 hooks 目錄 | 依賴已安裝的 `sesshu` 套件 |

---

## 4. `commands/`

`commands/sesshu-config.md` — `/sesshu-config` 的 prompt 檔。
含 18 個變數的型別與有效值表、List Mode 顯示格式、Set Mode 驗證規則與錯誤訊息。
`__SESSHU_ENV_PATH__` 佔位符在安裝時被替換。

`src/sesshu/assets/commands/sesshu-config.md` 是 pip 打包用的同份副本。

---

## 5. `scripts/`

| 檔案 | 用途 | 主要函式 |
|---|---|---|
| `benchmark_suite.py` | 產生 benchmark 執行計畫、彙總 JSONL 結果、評估 H1–H4 假說 | `discover_tasks()` `build_plan()` `read_jsonl()` `aggregate_results()` `evaluate_hypotheses()` |
| `measure_latency.py` | 量測 OpenAI 相容 endpoint 延遲 | `call_once()` `_env_default()`（支援 `SESSHU_*` 與 legacy `CC_LM_*`） |
| `update_json_config.py` | 被 `install.ps1` 呼叫，更新 `settings.json` / `hooks.json` | `is_runtime_sesshu_hook()` `clean_entries()` `update_settings()` |

`scripts/__init__.py` 存在，所以測試用 `from scripts import benchmark_suite` 直接 import，
**不用 `importlib.util` 動態載入**（`CLAUDE.md`「Testing」、`AGENTS.md:175`）。

`benchmark_suite.py` 的三個 group（`benchmarks/README.md`）：
`baseline`（停用 hook）、`sesshu-accepted`（接受 `/clear`）、`sesshu-ignored`（不 `/clear`）。
預設計畫是 4 tasks × 3 groups × 5 runs。實際執行**刻意手動**，
因為 group B 需要人在正確時機做 `/clear` 決策。

---

## 6. `tests/`

`tests/conftest.py` 提供兩個共用件：

- `run_hook()` helper
- `fake_lm_server` fixture — 起一個隨機 port 的 `ThreadingHTTPServer`
  （任何走 LM 呼叫路徑的測試都該用它）

30 個測試檔，大致一個模組一檔：

```
test_core.py            test_config_io.py       test_io.py
test_stop_hook.py       test_start_hook.py      test_precompact_hook.py
test_adapters.py        test_gemini_adapter.py  test_copilot_adapter.py
test_lm.py              test_lm_response_size.py
test_structure.py       test_prompts.py         test_transcript.py
test_events.py          test_fallback.py
test_wiki.py            test_search.py          test_related.py
test_async_ingest.py    test_ingest_queue.py    test_ingest_worker.py
test_runtime.py         test_io_append_jsonl_errors.py
test_cli_install.py     test_install_uninstall.py  test_install_ps1.py
test_benchmark_suite.py test_measure_latency.py test_measure_latency_large_response.py
```

測試規範（`AGENTS.md:169-177`）：用 `tmp_path` 做檔案系統行為；
**不得改動真實的 Claude Code / Codex / Gemini CLI 設定或資料檔**。

---

## 7. `benchmarks/`

```
benchmarks/README.md          groups、run plan、結果欄位格式
benchmarks/tasks/
    T1-multi-file-c-refactor.md
    T2-zephyr-rtos-debug.md
    T3-sva-to-cocotb.md
    T4-spec-to-code-register-map.md
benchmarks/results/           完成的 JSONL（.gitignore 除 .gitkeep 外）
```

結果 JSONL 每行一個 run，欄位含
`group` `task_id` `run_index` `task_passed` `task_score` `turn_count`
`total_input_tokens` `clear_events` `summary_injections` `wall_time_sec`。

---

## 8. `docs/`

| 檔案 | 內容 |
|---|---|
| `smoke-test.md` | 真實 runtime 生命週期檢查清單（Claude Code 與 Codex 兩節），155 行 |
| `sesshu-architecture-and-usage-guide.html` | 單一 HTML 長文，涵蓋 Introduction、Architecture Overview、Core Flows、Runtime Adapters、Transcript Processing、LM Interaction & Validation、Wiki Persistence & Search、Fallback Path、Related-Session Detection、Module Reference、Testing & Debugging、Privacy & Security、Limitations & Extension Points、Performance Tuning |

---

**下一步** → [troubleshooting.md](troubleshooting.md)
