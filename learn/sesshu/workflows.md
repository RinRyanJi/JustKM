# 常見工作流程

> 回到 [知識區入口](_index.md) ｜ 上一篇 [installation-and-usage.md](installation-and-usage.md) ｜ 下一篇 [code-map.md](code-map.md)

本文所有時序皆依 `src/sesshu/core.py`、`src/sesshu/adapters/`、`src/sesshu/cli.py` 實際程式碼確認，未經確認處明確標註。

---

## 1. Workflow A — Stop（每輪結束，HITL mode）

```
assistant 回覆結束
   │
   ▼ runtime 觸發 Stop（Gemini 為 AfterAgent）
run_stop()                                        core.py:346
   │
   ├─ mode != "hitl"             → exit 0（auto mode 下 Stop 是 no-op）
   ├─ 讀 + 刪 ingest/status.json  → 待送的背景 ingest 錯誤訊息
   ├─ 讀 + 刪 related_notice.json → 待送的相關 session 通知
   ├─ stop_hook_active           → exit 0（防迴圈）
   ├─ prompts 空                 → exit 0
   │
   ├─ _prepare_ingest()                           core.py:159
   │     ingest_mode == "background" → spawn_ingest_worker()，立刻返回
   │     ingest_mode == "sync"（預設）→ record_stop_payload() + run_ingest()
   │
   ├─ 讀 transcript 大小
   │
   ├─ len(raw) < THRESHOLD ?
   │     有待送通知 → 只送通知
   │     SESSHU_VERBOSE=1 → 送 "below threshold; nothing to compress"
   │     否則 → exit 0（靜默）
   │
   ├─ seed.md 不存在 ? → 只送待送通知，否則 exit 0
   ├─ shown.flag 存在 ? → 已提示過；只送待送通知（VERBOSE 時多送 "compression pending"）
   │
   └─ 寫 shown.flag → 送 systemMessage
```

**去重機制**：`shown.flag` 在 `run_stop()` 送出提示時寫入（`core.py:467-473`），
在 `run_start()` 接受 `/clear` 時刪除（`core.py:670-681`）。
所以每個「壓縮週期」只會提示一次。

---

## 2. Workflow B — Ingest（壓縮的實際動作）

```
run_ingest()                                      core.py:188
   │
   ├─ mode != "hitl"        → skip
   ├─ stop_hook_active      → skip
   ├─ prompts 空 / transcript 不存在 / 讀失敗 → skip
   │
   ├─ 讀 ingest/cursor.json
   │     cursor.session_id != 目前 session_id → 重置 cursor 為 {}
   │
   ├─ 間隔 gate：now - last_ingest_ts < INGEST_MIN_INTERVAL → skip
   │
   ├─ target_end = min(len(raw), chars_processed + INGEST_CHUNK_CHARS)
   │  diff = raw[chars_processed : target_end]
   │  diff 為空 → skip
   │
   ├─ excerpt = diff[-MAX_INPUT:]
   ├─ events = extract_events(excerpt)
   │  structured_events = format_structured_events(events) or "(none)"
   │
   ├─ seed.md 存在 ? → prompts["stop"]["update"]（帶 {existing}）
   │                    否則 prompts["stop"]["create"]
   │
   ├─ call_lm_with_retry(config, prompt, REQUIRED_SECTIONS, max_retries=2)
   │     每次呼叫後 validate_sections()；缺段落就用 build_retry_prompt() 重試
   │     回傳「最後一次非空的輸出」，即使段落仍不完整
   │
   ├─ summary 為空（LM 失敗）:
   │     無 seed → build_fallback_page(events) 寫 wiki + seed
   │               ★ cursor 刻意不前進 → 下次用完整 diff 重試
   │               → 回傳 "ok"
   │     有 seed → 保留舊 seed，回傳 "lm_error"
   │
   ├─ extract_recent_turns(transcript, KEEP_TURNS)
   ├─ build_seed_content(summary, recent_turns, prompts)
   ├─ write_wiki_page() → wiki/sessions/*.md + 更新 FTS 索引 + index.md + log.md
   ├─ write_private_text(seed.md)
   ├─ 寫 cursor.json = {session_id, target_end, now}
   │
   └─ SESSHU_RELATED=1 → _run_related_detection()
```

**cursor 的三個關鍵性質**（**來源已確認**）：

1. **增量**：只送 `[chars_processed, target_end)` 這段新內容給 LM（`core.py:249-250`）
2. **分塊**：單次最多 `INGEST_CHUNK_CHARS`（預設 20000）字元
3. **session 綁定**：`session_id` 不符就整個重置（`core.py:235-238`）

---

## 3. Workflow C — SessionStart（`/clear` 之後）

```
使用者執行 /clear（或 runtime 等價事件）
   │
   ▼
run_start()                                       core.py:598
   │
   ├─ 可接受的 source：
   │     mode == "hitl" → {"clear"}
   │     mode == "auto" → {"compact", "clear"}
   │   不符 → exit 0
   │
   ├─ ingest.lock 存在 ?
   │     is_lock_stale(lock, timeout) → 刪掉搶回
   │     否則 → 輪詢（每 0.5s）最多 INGEST_WAIT_SECONDS 秒
   │             逾時仍在 → 記 debug，用現有 seed 繼續
   │
   ├─ 刪除 related_state.json + related_notice.json
   ├─ prompts 空 / seed 不存在 → exit 0
   │
   ├─ summary = seed.md 內容
   ├─ pages = select_wiki_pages(wiki_dir, summary)
   │     sync_missing_pages() → 對帳 FTS 索引（新增/修改/刪除）
   │     extract_keywords(summary) → 取 Goal/Current state/Tasks/Blockers 的前 20 詞
   │     search_wiki() → FTS5 MATCH，失敗或 0 結果退回 grep
   │     仍無 → 最近 5 頁
   │
   ├─ context = summary + build_wiki_hint(pages)
   ├─ SESSHU_VERBOSE=1 → notify = build_start_notification(len(pages))
   │     "sesshu: restored prior session context (3 related pages available for lookup)"
   │     count 為 0 時省略括號子句
   │
   ├─ adapter.build_start_output(context, notify)
   │
   └─ 刪除：seed.md、shown.flag、ingest/cursor.json、
            ingest/status.json、ingest/queue.jsonl
```

⚠️ **注意刪除清單**（`core.py:667-681`）：`ingest.lock` **不在**這份刪除清單裡；
它由 worker 的 `_release_lock()` 負責，或在下次 `run_start()` 判定為 stale 時清掉。

---

## 4. Workflow D — Auto mode（PreCompact / PreCompress）

```
runtime 判定要壓縮 conversation
   │
   ▼ 觸發 PreCompact（Gemini 為 PreCompress）
run_precompact()                                  core.py:478
   │
   ├─ mode != "auto" → exit 0
   ├─ prompts 空 / transcript 不存在 → exit 0
   │
   ├─ excerpt = raw[-MAX_INPUT:]     ★ 尾端切片，不用 cursor、不設間隔 gate
   ├─ events = extract_events(excerpt)
   ├─ seed 存在 ? update : create（與 HITL 相同的模板）
   ├─ call_lm_with_retry()
   │     失敗且無 seed → 寫 fallback page
   │     失敗且有 seed → 保留舊 seed
   ├─ extract_recent_turns() → build_seed_content()
   ├─ 寫 wiki page + seed.md    ★ 不寫 cursor、不做 related 偵測
   │
   └─ return 0, ""               ★ 靜默，無 systemMessage
   │
   ▼ runtime 完成 compaction，觸發 SessionStart source="compact"
run_start() 注入 seed 到剛壓縮完的 session，然後刪除 seed
```

**HITL vs Auto 決策表**：

| 你的情境 | 建議 mode |
|---|---|
| 互動式開發，想自己控制何時重置 | `hitl`（預設） |
| 無人值守 / 批次執行，不能有人按 `/clear` | `auto` |
| 想要「完全不打擾」的體驗 | `auto` |
| 想保留完整的增量 ingest 與相關 session 偵測 | `hitl`（auto 不做這兩件事） |

---

## 5. Workflow E — 背景 ingest（`SESSHU_INGEST_MODE=background`）

```
run_stop() → _prepare_ingest() → spawn_ingest_worker()   async_ingest.py:162
   │
   ├─ _record_stop_payload()（Copilot 專用）
   ├─ _enqueue_transcript_range() → 追加一筆 QueueRecord 到 ingest/queue.jsonl
   ├─ SESSHU_INGEST_SYNC=1 → _run_inline() 直接跑完（測試用）
   ├─ 間隔 gate：距上次 ingest < MIN_INTERVAL → 不 spawn
   ├─ 原子建鎖 ingest.lock
   │     os.open(O_CREAT|O_EXCL|O_WRONLY, 0o600)
   │     內容 {session_id, pid, token, state, ts}
   │     FileExistsError → 已有 worker
   │     其他 OSError → 重試 3 次（0.05s / 0.10s backoff）
   └─ spawn ingest_worker.py 子行程 → Stop hook 立刻返回

ingest_worker.py main()                          ingest_worker.py:89
   ├─ _queue_payloads() → 取出自己 runtime 的第一筆記錄
   ├─ _run_ingest_returning_status() → True / False / None
   ├─ _update_queue()
   │     True  → 從 queue 移除
   │     False → attempts+1；>= INGEST_MAX_RETRIES 則移除，否則留著
   │     None  → 不動（正常 skip）
   ├─ True  → 刪 status.json
   │  False → 寫 status.json {"status":"error","message":"sesshu: background ingest failed (LM unavailable). Will retry."}
   │  例外  → 寫 status.json {"status":"error","message":"sesshu: worker crashed: ..."}
   └─ finally: _release_lock(lock, token)   ← 比對 token 才刪
```

`status.json` 是**一次性**的：`run_stop()` 讀完立刻 `unlink()`（`core.py:361-376`），
訊息會併進下一則 `systemMessage` 給使用者。

---

## 6. Workflow F — 相關 session 偵測（`SESSHU_RELATED=1`，opt-in）

```
run_ingest() 成功寫入 seed 之後 → _run_related_detection()   core.py:80-141
   │
   ├─ 讀 related_state.json；session_id 不符則重置
   ├─ own_pages += 本次剛寫的 page path（排除自己的頁）
   ├─ find_related(wiki_dir, seed, exclude=own_pages, threshold)   related.py:33
   │     extract_keywords(seed) → search_wiki(limit=10)
   │     逐頁 score_related() = 命中的相異關鍵字數 / 總相異關鍵字數（0.0–1.0）
   │     >= SESSHU_RELATED_THRESHOLD 才留下，依分數排序
   ├─ decide_notifications(hits, state)                          related.py:93
   │     未曾 surfaced 過 → 通知
   │     曾 surfaced 但分數上升超過 EPSILON(0.1) → 再通知
   │     否則 → 不通知
   └─ 有通知 → 寫 related_notice.json
        {"message": "sesshu: related prior session detected — <相對路徑> (relevance: 0.42)"}
        並把該頁記進 surfaced
   → 寫回 related_state.json {session_id, surfaced, own_pages}

下一次 run_stop() → 讀 related_notice.json → 立刻刪除 → 併入 systemMessage
```

**通知內容的限制**（`AGENTS.md:167`）：只能含簡短事實陳述、wiki 頁相對路徑、
正規化相關性分數；**不得含摘要內容或操作指示**。

---

## 7. 跨 runtime 整合對照（**來源已確認**）

### 7.1 事件與設定檔

| | Claude Code | Codex | Gemini CLI | Copilot CLI |
|---|---|---|---|---|
| Stop 事件 | `Stop` | `Stop` | **`AfterAgent`** | `Stop` |
| 預壓縮事件 | `PreCompact` | `PreCompact` | **`PreCompress`** | `PreCompact` |
| Start 事件 | `SessionStart` | `SessionStart` | `SessionStart` | `SessionStart` |
| 設定檔 | `settings.json` | **`hooks.json`** | `settings.json` | `settings.json` |
| 設定目錄 | `~/.claude` | `~/.codex` | `~/.gemini` | `~/.copilot` |
| timeout 欄位 | `timeout: 120`（秒） | `timeout: 120` | `timeout: 120000`（**毫秒**） | `timeoutSec: 120` |
| SessionStart matcher | `clear` + `compact` | `clear` + `compact` | **無 matcher** | **無 matcher** |
| `/sesshu-config` | ✅ | ✅ | ✅ | ❌ |
| transcript 來源 | `.jsonl` 檔 | `.jsonl` 檔 | `.jsonl` 檔 | **stdin payload → pseudo-transcript** |
| auto mode | ✅ | ✅ | ✅ | ✅ |

來源：`src/sesshu/cli.py:197-267`（四個 `_install_*` 函式）、`src/sesshu/adapters/`

### 7.2 Copilot CLI 的三個特殊處理

**【來源已確認】**（`src/sesshu/adapters/copilot.py`）

1. **Pseudo-transcript**（`copilot.py:21-72`）
   Copilot 不寫 `.jsonl`，而是把 stop 事件當 JSON 從 stdin 傳入。
   `record_stop_payload()` 把整個 payload 序列化成一行
   `{"type":"raw","role":"assistant","content":"<json>","timestamp":...}`，
   追加到 `<data_dir>/copilot_<sha256(session_id)>.jsonl`。
   之後 `run_ingest()` 就能像讀其他 transcript 一樣讀它。
   `core.py:171-176` 的長註解解釋了這個設計。

   ⚠️ **順序很重要**：`_prepare_ingest()` 在同步模式下**必須先** `record_stop_payload()`
   再 `run_ingest()`，否則 `request.transcript_path` 指向的檔案還不存在。
   `CHANGELOG.md`（Unreleased/Fixed）記錄了修這個問題的 commit：
   「Restore Copilot stop message emission by materializing pseudo-transcript before ingest gating」。

2. **source 正規化**（`copilot.py:12-14, 77-83`）
   `{"new", "startup", "resume"}` 三種 source 一律映射成 `"clear"`。
   註解說明理由：Copilot 沒有 `/clear` 指令，每個新 agent session 都等同於
   Claude Code 的 post-`/clear` SessionStart。

   > **【推論】**：這意味著在 Copilot 上，**每次啟動 CLI（含 `resume`）都會注入 seed**，
   > 而不像 Claude Code 只在明確 `/clear` 後注入。這是行為上的實質差異，
   > 來源文件未特別提醒。

3. **輸出格式**（`copilot.py:85-88`）
   用 `{"decision": "block", "reason": <訊息>}` 而非 `{"systemMessage": ...}`；
   訊息為空時送 `{"decision": "allow"}`。
   `build_start_output()` 的 `notify` 參數被忽略（Copilot 無等價機制）。

### 7.3 遷移舊資料

**【來源已確認】**（`README.md:489-495`、`docs/smoke-test.md:146-154`）

`~/.claude/sesshu` 的舊資料**不會自動遷移**。手動搬：

```bash
mkdir -p ~/.sesshu && cp -R ~/.claude/sesshu/. ~/.sesshu/
```

末尾的 `/.` 是關鍵——複製「目錄內容」而非目錄本身，
避免在目的地已存在時變成 `~/.sesshu/sesshu`。

---

## 8. Smoke test 流程（**來源已確認**，`docs/smoke-test.md`）

來源提供一份真實 runtime 檢查清單。通過條件（兩個 runtime 相同）：

> Stop hook 建立 `seed.md` 與 `shown.flag`；`/clear` 之後 SessionStart hook
> 注入壓縮後的 context 並刪除這兩個暫存檔。

步驟摘要：

1. 安裝該 runtime 的 hook
2. 設低門檻（`SESSHU_THRESHOLD=1000`、`SESSHU_MAX_INPUT=4000`、`SESSHU_KEEP_TURNS=2`）
3. 跑到超過門檻 → 檢查：runtime 顯示 Sesshu 訊息、`~/.sesshu/seed.md` 存在、
   `~/.sesshu/shown.flag` 存在、`~/.sesshu/wiki/sessions/` 有 session 頁
4. 執行 `/clear` → 檢查：新 session 收到壓縮 context、內容含最近輪次與 wiki hint、
   `seed.md` 與 `shown.flag` 被刪除、**舊的原始 transcript 沒有被重播進新 context**
5. （選用）暫時把 SessionStart 指令包一層 `tee` 記錄 stdin，確認輸入含 `"source":"clear"`，
   測完還原

Codex 額外注意（`docs/smoke-test.md:93-95`）：從你要測的專案啟動 Codex，
執行 `/hooks` 並在 Codex 詢問時信任 Sesshu hook——
**repo-local Codex hook 需要專案 hook 層被信任後才會執行**。

> 本知識區**未執行**任何 smoke test，以上為來源文件的步驟轉述。

---

**下一步** → [code-map.md](code-map.md)
