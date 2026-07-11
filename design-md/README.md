# DESIGN.md 設計系統圖鑑

> 來自 [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md)（100k+ stars）— 74 個品牌的設計規格全集。

## 📂 目錄結構

```
design-md/
├── index.html          # 設計展示頁（品牌卡片 + 篩選 + Modal 預覽）
├── brands.json         # 74 個品牌的色票資料（自動產生）
└── library/            # 原始 DESIGN.md 規格檔
    ├── stripe/
    │   ├── DESIGN.md   # Stripe 設計規格（24KB, 完整色票/字型/元件）
    │   └── README.md
    ├── linear.app/
    ├── apple/
    ├── vercel/
    └── ... (共 74 個品牌)
```

## 🚀 線上瀏覽

https://rinryanji.github.io/JustKM/design-md/

## 🎨 涵蓋品牌（74 個）

**科技 / 平台**：Stripe, Vercel, Linear, Notion, Figma, Framer, Slack, GitHub, Raycast, Webflow, Warp, Sentry, Sanity, Supabase, Mintlify, Replicate, The Verge, Wired, SpaceX, PlayStation, Nintendo, Spotify, Uber, Pinterest, Expo, MongoDB, ClickHouse, HashiCorp, MasterCard, Vodafone, Wise, Revolut, Shopify, Resend, Clay, Intercom, PostHog, Miro, Airtable

**AI**：Claude, Minimax, Mistral, Cohere, Together.ai, Replicate, RunwayML, Voltagent, OpenCode.ai, Lovable, Cursor, X.AI, Ollama, ElevenLabs

**車廠**：BMW, BMW-M, Bugatti, Ferrari, Lamborghini, Renault, Tesla

**金融**：Stripe, Binance, Coinbase, Kraken, MasterCard, Wise, Revolut

**設計工具**：Figma, Framer, Miro, Clay, Linear, Notion, Webflow, Raycast

**消費品牌**：Airbnb, Apple, Nike, Starbucks, HP, IBM, Dell-1996, Meta, NVIDIA

## 📄 DESIGN.md 格式範例（Stripe）

每個 DESIGN.md 是結構化 YAML 規格，包含：

- **`name`** / **`description`** — 設計語言概述
- **`colors`** — 完整色票（primary、ink、canvas、hairline、accent 等）
- **`typography`** — 字型系統（display / heading / body / caption）
- **`spacing`** / **`radii`** / **`shadows`** — 設計 token
- **`buttons`** / **`cards`** / **`inputs`** — 元件規格

可直接餵給 Claude Code / Cursor 等 AI 編碼助手：

```bash
# 範例：在 Cursor 中
"把這個 DESIGN.md 套用到我的 React 元件"

# 或把整個目錄丟給 Claude：
cat design-md/library/stripe/DESIGN.md
> "請用這個設計規格生成一個 landing page"
```

## 📝 授權

原始 repo 由 VoltAgent 維護，本圖鑑僅為索引與展示，原始 DESIGN.md 內容版權歸各原品牌所有。