# 🗂️ JustKM — 我的私人知識庫

> 一個統一的收納入口。未來各主題放子資料夾，互不干擾。

## 🔗 快速連結（線上瀏覽）

> 全站以 **GitHub Pages** 部署，來源分支 `main`。以下連結需該內容已合併進 `main` 後才會生效。

- 🏠 **網站首頁**：<https://rinryanji.github.io/JustKM/>
- 🎓 **AI 學習總目錄（學習區入口）**：<https://rinryanji.github.io/JustKM/learn/>
  - 🗜️ AI 對話/上下文壓縮實戰（跨 Agent 通用）：<https://rinryanji.github.io/JustKM/learn/ai-context-compression.html>
  - 🏢 **NEW · Palantir 如何一站式解決資料碎片化（SVG 圖解／簡約明亮）**：<https://rinryanji.github.io/JustKM/learn/palantir-data-fragmentation.html>
  - 🧠 訓練小模型做單一任務（SLM 專用化）：<https://rinryanji.github.io/JustKM/learn/train-slm-single-task.html>
  - 🧰 Matt Pocock AI Skills 總覽（系列入口）：<https://rinryanji.github.io/JustKM/learn/matt-pocock-skills.html>
    - ✍️ AI 寫作系統：<https://rinryanji.github.io/JustKM/learn/matt-pocock-writing-skills.html>
    - 🏭 AI 程式開發流程：<https://rinryanji.github.io/JustKM/learn/matt-pocock-dev-workflow.html>
  - 🥧 拆解 oh-my-pi（omp）—— Rust 的 terminal-first AI 編碼 Agent：<https://rinryanji.github.io/JustKM/learn/oh-my-pi.html>
  - 📊 **NEW · Codex 使用量計算原理（token / 成本 / 配額怎麼算出來的）**：<https://rinryanji.github.io/JustKM/learn/codex-usage-calculation.html>
  - 📚 **NEW · Sesshu 知識區（session 壓縮 hook 套件）**：<https://rinryanji.github.io/JustKM/learn/sesshu/_index.md>
    - 📄 官方架構與使用指南（HTML 原檔）：<https://rinryanji.github.io/JustKM/learn/sesshu/sesshu-architecture-and-usage-guide.html>
    - 🗜️ **NEW · Sesshu 壓縮技術說明（繁中 HTML 原檔）**：<https://rinryanji.github.io/JustKM/learn/sesshu/sesshu-compression-explained.html>
- 🌐 網路小白入門（REST / WebSocket）：<https://rinryanji.github.io/JustKM/learn/network-basics.html>
- 🎨 74 品牌設計系統圖鑑：<https://rinryanji.github.io/JustKM/design-md/>
- 📈 美股交易教學：<https://rinryanji.github.io/JustKM/invest/course.html>

## 結構

```
JustKM/
├── README.md             ← 你在這裡
├── index.html            ← 入口（74 個設計模板 + 主題卡）
├── invest/               ← 美股交易教學（AutoInvest）
├── learn/                ← AI / ML 學習（SLM 訓練、網路基礎、Agent）
├── raw/                  ← 原始資料
├── design-md/            ← 74 個品牌 DESIGN.md
└── travel/               ← 旅遊行程
    └── ishigaki-2026/
        ├── index.html        ← 行李清單 (Binance 風格, 1358 行)
        └── itinerary.html    ← 行程總覽 (Sanity 風格, 1212 行)
```

## 各主題目錄

| 主題 | 路徑 | 狀態 | 說明 |
|---|---|---|---|
| 📈 美股交易 | [`invest/`](invest/course.html) | ✅ 已建立 | 日內 × 波段交易完整教學（8 章 + 速查表） |
| 🧳 石垣跳島 · 行李 | [`travel/ishigaki-2026/index.html`](travel/ishigaki-2026/index.html) | ✅ 已建立 | Binance 風格 · 14 段含 SVG 時間軸 · 2人 6D5N |
| 🗓️ 石垣跳島 · 行程 | [`travel/ishigaki-2026/itinerary.html`](travel/ishigaki-2026/itinerary.html) | ✅ 重列 | Sanity 風格 · 兩段住宿 + 颱風應變 SOP |
| 🎨 74 品牌 DESIGN.md | [`design-md/`](design-md/index.html) | ✅ 已建立 | Stripe/Linear/Vercel/Apple/Cursor 等設計 token |
| 🤖 Pi Agent 最佳實踐 | [`learn/ai-agents/pi-agent-best-practices-2026-06.md`](learn/ai-agents/pi-agent-best-practices-2026-06.md) | 🟡 metadata only | 暮閒 2026-06 影片,無字幕,待 Briefing Doc 補強 |
| 🎓 AI 學習總目錄 | [`learn/`](learn/index.html) | ✅ 已建立 | AI/ML 學習區入口 · 收整下列所有單元，分區導覽 |
| 🗜️ AI 對話/上下文壓縮實戰 | [`learn/ai-context-compression.html`](learn/ai-context-compression.html) | ✅ 已建立 | 跨 Agent 通用 · Claude Code/Codex 對照 + 四層次 + 通用策略 + GitHub 專案來源 |
| 🏢 Palantir 解決資料碎片化 | [`learn/palantir-data-fragmentation.html`](learn/palantir-data-fragmentation.html) | ✅ 已建立 | 簡約明亮 · 5 張 SVG 圖解 · Foundry/Ontology/AIP/Apollo · 閉環寫回 |
| 🥧 拆解 oh-my-pi（omp） | [`learn/oh-my-pi.html`](learn/oh-my-pi.html) | ✅ 已建立 | Rust terminal-first AI 編碼 Agent（Pi 的 fork）· 架構/31 工具/角色路由/亮點 |
| 📊 Codex 使用量計算原理 | [`learn/codex-usage-calculation.html`](learn/codex-usage-calculation.html) | ✅ 已建立 | 9 個觀念 · 用例子講不用公式 · cached 扣抵／三種百分比／週配額反推／去重匯入／誤差來源 |
| 🧠 訓練小模型做單一任務 | [`learn/train-slm-single-task.html`](learn/train-slm-single-task.html) | ✅ 已建立 | SLM 專用化實戰 · 13 節 · LoRA/QLoRA + 蒸餾 + NVIDIA 六步 · 含決策樹與程式範例 |
| 🧰 Matt Pocock Skills 總覽目錄 | [`learn/matt-pocock-skills.html`](learn/matt-pocock-skills.html) | ✅ 已建立 | 系列入口 · 兩篇深入拆解 + 整個 repo 的 Skill 速查地圖 |
| ✍️ Matt Pocock AI 寫作系統 | [`learn/matt-pocock-writing-skills.html`](learn/matt-pocock-writing-skills.html) | ✅ 已建立 | 拆解 mattpocock/skills 的 fragments/shape/beats · 探索 vs 開採 · 含手動版流程 |
| 🏭 Matt Pocock AI 程式開發流程 | [`learn/matt-pocock-dev-workflow.html`](learn/matt-pocock-dev-workflow.html) | ✅ 已建立 | grill→spec→tickets→implement(TDD)→review 生產線 · 17 個工程 Skill · 含手動版 |
| 📚 Sesshu 知識區 | [`learn/sesshu/_index.md`](learn/sesshu/_index.md) | ✅ 已建立 | session 歷史壓縮 hook 套件（Claude Code/Codex/Gemini/Copilot CLI）· 7 份中文導覽 + [官方 HTML 原檔](learn/sesshu/sesshu-architecture-and-usage-guide.html) + [壓縮技術說明](learn/sesshu/sesshu-compression-explained.html) |

---

## 如何使用

- 每個主題是一個獨立子資料夾，放該主題的所有檔案（HTML / Markdown / 圖）
- 主題頁面之間不強耦合，可獨立維護
- 主題內連接與跨主題連結都歡迎，只要寫得清楚
- 字級調整器：每頁都有 `Cmd/Ctrl + / - / 0` 快捷鍵，跨頁 localStorage 同步

## 部署

本站用 **GitHub Pages** 部署：
- 來源：`main` branch
- 路徑：根目錄 → https://rinryanji.github.io/JustKM/

> ⚠️ **注意**：Pages 只服務 `main` 分支的內容。若在其他分支（如 `claude/*`）新增頁面，
> 線上網址會顯示 404，**必須先把分支合併進 `main`**，Pages 才會自動重建並生效（約 1 分鐘）。

---

最後更新：2026-08-06（新增 learn/sesshu/ — Sesshu 知識區入口，已收進首頁與學習區導覽）
