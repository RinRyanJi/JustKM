# 🗂️ JustKM — 我的私人知識庫

> 一個統一的收納入口。首頁只放主題 hub；深入文章收在各區總目錄（zone index）。

## 🔗 快速連結（線上瀏覽）

> 全站以 **GitHub Pages** 部署，來源分支 `main`。以下連結需該內容已合併進 `main` 後才會生效。

- 🏠 **網站首頁（主題 hub）**：<https://rinryanji.github.io/JustKM/>
- 💹 **投資與程式交易**：首頁卡片 → [`invest/`](https://rinryanji.github.io/JustKM/invest/course.html)
- 🎓 **AI 學習總目錄（學習區入口）**：<https://rinryanji.github.io/JustKM/learn/>
  - 分組：AI 工作流 & Skills · 上下文／記憶（含 Sesshu）· 模型訓練 & Agent · 語音／嵌入式 · 工程實踐／職場 · 開發基礎 · 其他
  - 📚 Sesshu 知識區：<https://rinryanji.github.io/JustKM/learn/sesshu/_index.md>
  - 🏢 精選 · Palantir 資料碎片化：<https://rinryanji.github.io/JustKM/learn/palantir-data-fragmentation.html>
  - 🛡️ 精選 · 讓工程師擁抱 AI：<https://rinryanji.github.io/JustKM/learn/engineers-adopt-ai.html>
- 🧩 **Skills 分析專區**：<https://rinryanji.github.io/JustKM/skills/>
- 🎨 **設計系統圖鑑（74 品牌）**：<https://rinryanji.github.io/JustKM/design-md/>
- ✈️ **旅遊 · 八重山**：<https://rinryanji.github.io/JustKM/travel/ishigaki-2026/>
- ⚖️ **法律 · 蝦皮租約**：<https://rinryanji.github.io/JustKM/legal/shopee-lease-review.html>
- 🗂️ **原始資料**：[`raw/DayTrade/`](https://rinryanji.github.io/JustKM/raw/DayTrade/) · [`raw/Survery/`](https://github.com/RinRyanJi/JustKM/tree/main/raw/Survery)

## 結構（入口層級）

```
JustKM/
├── README.md
├── index.html              ← Level 1：主題 hub 首頁（投資／AI／Skills／設計／旅遊／法律／原始資料）
├── invest/                 ← 美股交易與券商 API
├── learn/
│   └── index.html          ← Level 2：AI 學習總目錄（分組索引 → 各篇文章）
├── skills/
│   └── index.html          ← Level 2：Skills 分析專區
├── design-md/              ← 74 個品牌 DESIGN.md
├── travel/ishigaki-2026/   ← 旅遊知識庫
├── legal/                  ← 契約／法務
└── raw/                    ← 原始資料（未整理）
```

**導覽原則**

1. **首頁**只放各主題 hub（及最多 1–3 張 AI 精選卡），不堆 40+ 文章卡。
2. **深入文章**請從 `learn/index.html` / `skills/index.html` 等 zone index 進入。
3. Sesshu 子頁只掛在學習區「上下文／記憶」群組，不再出現在首頁。

## 各主題目錄

| 主題 | 路徑 | 狀態 | 說明 |
|---|---|---|---|
| 🏠 首頁（hub） | [`index.html`](index.html) | ✅ | 七大主題入口；AI 區 = 學習總目錄 hub + 最多數張精選 |
| 📈 美股交易 | [`invest/`](invest/course.html) | ✅ | 日內 × 波段教學 + Alpaca／IBKR／Finnhub API 速查 |
| 🎓 AI 學習總目錄 | [`learn/`](learn/index.html) | ✅ | 分組導覽：工作流、上下文／記憶、Agent、語音、職場、基礎 |
| 🧩 Skills 分析專區 | [`skills/`](skills/index.html) | ✅ | 技能包拆解 · 6 維框架 + 五大類巡覽 |
| 🎨 74 品牌 DESIGN.md | [`design-md/`](design-md/index.html) | ✅ | Stripe／Linear／Vercel／Apple／Cursor 等設計 token |
| 🏝️ 八重山旅行知識庫 | [`travel/ishigaki-2026/`](travel/ishigaki-2026/index.html) | ✅ | 石垣 × 西表 × 竹富 · 行程／住宿／交通／飲食 |
| 📑 蝦皮租約 | [`legal/shopee-lease-review.html`](legal/shopee-lease-review.html) | ✅ | 房東逐條解析 · 附[一頁決策摘要](legal/shopee-lease-decision.html) |
| 📚 Sesshu 知識區 | [`learn/sesshu/_index.md`](learn/sesshu/_index.md) | ✅ | session 壓縮 hook · 經 learn 總目錄進入 |
| 🗂️ 原始資料 | [`raw/`](raw/DayTrade/index.html) | ✅ | DayTrade 手冊、美股 API 調研、自動化專案計畫 |

> 學習區內個別文章清單以 [`learn/index.html`](learn/index.html) 為準，此表不再逐篇列舉，避免與 zone index 重複維護。

---

## 如何使用

- 每個主題是一個獨立子資料夾，放該主題的所有檔案（HTML / Markdown / 圖）
- 主題頁面之間不強耦合，可獨立維護
- 新增深入文章：放到對應 zone，並在該區 `index.html` 掛卡；**不要**再往首頁堆卡
- 字級調整器：每頁都有 `Cmd/Ctrl + / - / 0` 快捷鍵，跨頁 localStorage 同步

## 部署

本站用 **GitHub Pages** 部署：
- 來源：`main` branch
- 路徑：根目錄 → https://rinryanji.github.io/JustKM/

> ⚠️ **注意**：Pages 只服務 `main` 分支的內容。若在其他分支（如 `claude/*`）新增頁面，
> 線上網址會顯示 404，**必須先把分支合併進 `main`**，Pages 才會自動重建並生效（約 1 分鐘）。

---

最後更新：2026-09-20（入口 IA 重整：首頁改為主題 hub；AI 文章牆下沉至 `learn/index.html` 分組索引；Skills 獨立成 Level 1；README 對齊真實路徑）
