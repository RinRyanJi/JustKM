# 🗂️ JustKM — 我的私人知識庫

> 一個統一的收納入口。未來各主題放子資料夾，互不干擾。

## 結構

```
JustKM/
├── README.md             ← 你在這裡
├── index.html            ← 入口（74 個設計模板 + 主題卡）
├── invest/               ← 美股交易教學（AutoInvest）
├── learn/                ← （未來）AI / ML 學習
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

---

最後更新：2026-07-11（新增 itinerary.html）
