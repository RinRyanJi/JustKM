# Sesshu 知識區入口

> **來源專案**：`D:/AiWorkSpace/Sesshu-main`（Sesshu v0.1.0，Apache-2.0，Nexplico Ltd.）
> **本知識區位置**：`learn/sesshu/`
> **建立日期**：2026-08-06
> **語言**：繁體中文（技術名詞保留英文）

---

## 這是什麼

Sesshu（摂取）是一套給 **Claude Code / Codex CLI / Gemini CLI / GitHub Copilot CLI** 用的
**session 歷史壓縮 hook 套件**。它在對話變長時，透過一個 OpenAI 相容的 LM endpoint
（預設指向本機 Ollama）把 transcript 壓成一份結構化的 wiki 摘要，
並在你 `/clear` 後把摘要重新注入新 session。

本知識區把來源專案的 `README.md`、`AGENTS.md`、`CLAUDE.md`、`docs/`
以及 `src/`、`hooks/`、`commands/`、`scripts/` 的程式碼結構，整理成中文導覽。

---

## 閱讀順序

| # | 文件 | 適合誰 | 內容 |
|---|---|---|---|
| 1 | [README.md](README.md) | 所有人 | Sesshu 是什麼、解決什麼問題、適用/不適用情境 |
| 2 | [architecture.md](architecture.md) | 想理解設計的人 | 架構圖、核心元件、資料流、Hook/Command/Adapter 關係 |
| 3 | [installation-and-usage.md](installation-and-usage.md) | 要實際安裝的人 | Windows / Linux 安裝、設定觀念、日常使用 |
| 4 | [workflows.md](workflows.md) | 要理解執行時序的人 | session start/stop、compact、四種 runtime 的整合差異 |
| 5 | [code-map.md](code-map.md) | 要改程式的人 | 各目錄用途、重要檔案導覽 |
| 6 | [source-index.md](source-index.md) | 要讀原始碼的人 | 程式碼索引、建議閱讀順序、關鍵片段 |
| 7 | [troubleshooting.md](troubleshooting.md) | 遇到問題的人 | 常見問題、診斷方式、已知限制 |

**最短路徑**：只想知道「這東西值不值得裝」→ 讀 1；決定要裝 → 讀 3；裝完怪怪的 → 讀 7。

---

## 原始文件（未經翻譯／改寫）

| 文件 | 格式 | 說明 |
|---|---|---|
| [sesshu-architecture-and-usage-guide.html](sesshu-architecture-and-usage-guide.html) | HTML（英文） | 來源專案 `docs/` 的官方架構與使用指南原檔，**逐位元組原樣複製**（UTF-8 無 BOM，36,400 bytes） |
| [sesshu-compression-explained.html](sesshu-compression-explained.html) | HTML（繁中） | 來源專案 `docs/` 的**壓縮技術說明**原檔，**逐位元組原樣複製**（UTF-8 無 BOM，53,960 bytes，8 張內嵌 SVG） |
| [manual.html](manual.html) | HTML（繁中） | 來源專案 `docs/` 的**使用手冊**原檔，**逐位元組原樣複製**（UTF-8 無 BOM，35,713 bytes，5 張內嵌 SVG） |

**《Sesshu 壓縮技術：核心觀念與方法》** 是來源專案作者用繁體中文撰寫的長文，
專講「壓縮」這件事本身：為什麼對話會因 context 塞滿而「死掉」、四個核心觀念
（卸載而非刪減、游標式增量攝取、固定 12 節結構、分層降級）、資料如何一層層變小、
三個時間點的完整生命週期、wiki 讀回來為何不會再次塞爆、三條不可違反的紀律、
常見誤解、參數速查，以及與原始碼的對照表。

> 2026-08-06 更新：新增「精確一點說：每次到底送出什麼」小節（拆解 `excerpt` /
> `existing` / `structured_events` 三個提示詞成分的大小上限），以及 cursor 的
> 完整說明 —— 誤會澄清框、`cursor.json` 三個欄位的意義，與一張「單一 session 內
> 游標如何隨每次攝取前移」的時間軸圖。圖數由 7 張增為 8 張。

**《Sesshu 使用手冊》** 是同一位作者撰寫的操作導向文件，不談原理只講怎麼用：
安裝三步驟（pip 與原始碼兩種路徑，含 PowerShell 與 Bash 指令）、安裝後檔案落點、
LM endpoint 設定（Ollama 本機／雲端）、安裝成功檢查清單、hitl 與 auto 兩種模式的
日常操作、`/sesshu-config` 用法、依情境調參數對照表、疑難排解決策樹、解除安裝。

> 三份 HTML 的分工：**要裝要用** → `manual.html`；**想懂原理** →
> `sesshu-compression-explained.html`；**要完整技術參考** →
> `sesshu-architecture-and-usage-guide.html`（英文）。

這份英文 HTML 是來源專案作者撰寫的單一長文，涵蓋 Introduction、Problem & Motivation、
Architecture Overview、Operating Principles、Core Flows、Runtime Adapters、Configuration、
Trusted Config/Data Directories、Transcript Processing、Structured Event Extraction、
LM Interaction & Validation、Wiki Persistence & Search、Fallback Path (LM-Free)、
Related-Session Detection、Module Reference、Testing & Debugging、Privacy & Security、
Limitations & Extension Points 等章節。

> 上表 1–7 的中文文件是**導覽與再整理**；這份 HTML 是**一手來源**。
> 兩者說法不一致時，以 HTML 原檔與實際程式碼為準。
> 用瀏覽器開啟即可閱讀（純靜態，無外部資源相依）。

---

## 快速事實表（皆為來源已確認）

| 項目 | 值 | 來源 |
|---|---|---|
| 語言 / 版本 | Python 3.9+ | `pyproject.toml:10` |
| 第三方相依 | **無**（runtime 只用標準函式庫） | `pyproject.toml:35` |
| 授權 | Apache-2.0 | `pyproject.toml:11`、`LICENSE` |
| 版本 | 0.1.0 | `pyproject.toml:7` |
| 支援 runtime | claude / codex / gemini / copilot | `src/sesshu/cli.py:11` |
| CLI 入口 | `sesshu = "sesshu.cli:main"` | `pyproject.toml:44` |
| 測試框架 | pytest（30 個 test 檔） | `pyproject.toml:61-64`、`tests/` |
| 預設 LM | `http://localhost:11434/v1` + `qwen2.5:7b` | `src/sesshu/config.py:37-38` |

---

## 本知識區的標註慣例

- **【已確認】** — 直接來自來源檔案，並附相對路徑（例如 `src/sesshu/core.py:188`）。
- **【推論】** — 由程式碼行為推導、但來源文件未明說的結論。
- **【文件不一致】** — 來源專案自己的 Markdown 文件與實際程式碼有出入的地方。

> ⚠️ 本知識區**不含**任何 `.env` 實際內容、API key、token 或憑證。
> `.env.example` 只用來確認變數「名稱」，未複製任何值。

---

## 未複製到 JustKM 的內容

以下刻意留在來源專案，本知識區只做導覽不做複製：

- 完整原始碼（`src/sesshu/**/*.py`，約 20 個模組）
- 測試檔（`tests/test_*.py`，30 個）
- `install.sh` / `install.ps1` 完整腳本（僅摘要行為）
- `docs/smoke-test.md`
- `benchmarks/tasks/*.md`（4 份 benchmark 任務 prompt）

> 註：`docs/sesshu-architecture-and-usage-guide.html` 與
> `docs/sesshu-compression-explained.html` 原本列在此清單，
> 已於 2026-08-06 改為原樣複製進本目錄，見上方「原始文件」段落。
