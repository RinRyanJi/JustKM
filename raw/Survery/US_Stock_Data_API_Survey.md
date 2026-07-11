# 美股即時數據 API 調研報告 (2025-2026)

本文檔為自動化美股交易系統的數據源選型調研，涵蓋 12 個主要 API 平台的功能、價格、限制與適用場景。

---

## 快速對比表

| 平台 | 免費即時行情 | WebSocket | 逐筆數據 | Python SDK | 需券商帳戶 | 月費（即時） |
|------|:---:|:---:|:---:|:---:|:---:|---:|
| **Yahoo Finance (yfinance)** | ⚠️ 延遲15-20分 | ❌ | ❌ | yfinance | ❌ | 免費 |
| **Alpha Vantage** | ❌ | ❌ | ❌ | alpha_vantage | ❌ | $99.99 |
| **Finnhub** | ✅ | ✅ (50符號) | ❌ | finnhub-python | ❌ | 免費 |
| **Polygon.io** | ❌ | ✅ | ✅ (SIP) | polygon-api-client | ❌ | $199 |
| **IEX Cloud** | ❌ 已關閉 | — | — | — | — | — |
| **Alpaca Markets** | ✅ (IEX) | ✅ | ✅ (SIP $99) | alpaca-py | ❌ (免費帳戶) | 免費/$99 |
| **Interactive Brokers** | ✅ (SIP) | ✅ | ✅ Level 2 | ib_insync | ✅ | $10 (可減免) |
| **Tiingo** | ✅ (IEX) | ✅ | ❌ | — | ❌ | $30 |
| **Twelve Data** | ✅ | ❌ (付費才有) | ❌ | twelvedata | ❌ | $79 |
| **Charles Schwab** | ✅ (SIP) | ✅ | ✅ | schwab-py | ✅ | 免費 |
| **Tradier** | ✅ (SIP) | ✅ | ❌ | — | ✅ ($10/月) | $10 |
| **Nasdaq Data Link** | ❌ | ❌ | ❌ | nasdaqdatalink | ❌ | 免費(EOD) |

---

## 各平台詳細評測

### 1. Yahoo Finance (yfinance)
- **URL**: 透過 Python `yfinance` 套件（非官方）
- **免費方案**: 無限制，但資料延遲 15-20 分鐘
- **即時數據**: ❌ 延遲報價
- **數據類型**: OHLCV、基本面、選擇權鏈
- **傳輸方式**: REST（非官方 scraping）
- **Python 套件**: `pip install yfinance`
- **頻率限制**: 無官方限制，但過於頻繁會被封鎖
- **⚠️ 注意**: 非官方 API，違反 Yahoo TOS，經常因網站改版而壞掉。**不適合生產環境**
- **適合**: 學習用途、快速原型、歷史數據下載

### 2. Alpha Vantage
- **URL**: https://www.alphavantage.co/
- **免費方案**: 每日 25 次呼叫（非常受限）
- **付費方案**: $49.99/月 (75次/分) 起，即時數據 $99.99/月
- **即時數據**: 付費方案才有
- **數據類型**: OHLCV、技術指標、外匯、加密貨幣
- **傳輸方式**: REST
- **Python 套件**: `pip install alpha_vantage`
- **適合**: 需要技術指標計算的小量查詢

### 3. Finnhub ⭐ 免費首選
- **URL**: https://finnhub.io/
- **免費方案**: 60 次/分鐘 REST + WebSocket 即時串流（50 符號）
- **付費方案**: 依用量，企業級需洽詢
- **即時數據**: ✅ 美股即時報價（免費）
- **數據類型**: 即時報價、OHLCV、新聞、財報、ESG
- **傳輸方式**: REST + WebSocket
- **Python 套件**: `pip install finnhub-python`
- **適合**: 入門者免費即時數據首選，適合監控少量股票

### 4. Polygon.io (已更名 Massive)
- **URL**: https://polygon.io/
- **免費方案**: 延遲行情、5次/分鐘
- **付費方案**: $29/月(延遲) → $199/月(SIP即時) → $499/月(全量tick)
- **即時數據**: ✅ 付費方案，完整 SIP feed
- **數據類型**: 逐筆交易、報價、OHLCV、選擇權、外匯、加密
- **傳輸方式**: REST + WebSocket
- **Python 套件**: `pip install polygon-api-client`
- **適合**: 需要高品質 tick-level 數據的專業交易系統

### 5. IEX Cloud ❌ 已關閉
- 2024年8月31日關閉服務，已不可用

### 6. Alpaca Markets ⭐ 入門首選
- **URL**: https://alpaca.markets/
- **免費方案**: 免費帳戶 + 紙上交易 + IEX 即時行情
- **付費方案**: $99/月 SIP 全量數據
- **即時數據**: ✅ IEX 免費即時（覆蓋約 2% 成交量）；SIP 完整即時 $99/月
- **數據類型**: 報價、OHLCV（1分鐘到日線）、逐筆（SIP）
- **傳輸方式**: REST + WebSocket
- **Python 套件**: `pip install alpaca-py`
- **認證**: API Key + Secret
- **⚠️ 注意**: 免費 IEX 數據僅覆蓋約 2% 市場成交量，價格可能與實際有偏差
- **適合**: **入門者最佳選擇** — 免費紙上交易 + 數據 + 下單一體化

### 7. Interactive Brokers (IBKR) ⭐ 專業首選
- **URL**: https://www.interactivebrokers.com/
- **數據費用**: $10/月 SIP（月交易佣金 $30+ 可減免）
- **即時數據**: ✅ 完整 SIP + Level 2 深度行情
- **數據類型**: 報價、OHLCV、逐筆、Level 2、選擇權、期貨
- **傳輸方式**: TWS API（需運行 TWS 或 IB Gateway）
- **Python 套件**: `pip install ib_insync`
- **⚠️ 注意**: 需安裝並運行 TWS/IB Gateway；學習曲線較陡
- **適合**: **專業交易系統首選** — 數據最全、費用最低、支持真實下單

### 8. Tiingo
- **URL**: https://www.tiingo.com/
- **免費方案**: 有限 EOD 數據
- **付費方案**: $30/月（30+ 年歷史日線 + IEX 即時）
- **即時數據**: ✅ IEX 即時（2025年2月政策調整）
- **數據類型**: EOD、日內、新聞、基本面
- **傳輸方式**: REST + WebSocket
- **適合**: 回測用途（歷史數據性價比最高）

### 9. Twelve Data
- **URL**: https://twelvedata.com/
- **免費方案**: 8 次/分鐘，即時美股報價
- **付費方案**: $79/月起（WebSocket + 更高頻率）
- **即時數據**: ✅ 免費即時報價
- **數據類型**: OHLCV、100+ 技術指標
- **傳輸方式**: REST（免費）+ WebSocket（付費）
- **Python 套件**: `pip install twelvedata`
- **適合**: 需要內建技術指標計算的場景

### 10. Charles Schwab (原 TD Ameritrade)
- **URL**: https://developer.schwab.com/
- **費用**: 免費（需券商帳戶）
- **即時數據**: ✅ 完整 SIP 即時
- **數據類型**: 報價、OHLCV、選擇權鏈、Level 1
- **傳輸方式**: REST + WebSocket
- **Python 套件**: `pip install schwab-py`
- **⚠️ 注意**: OAuth token 每 7 天過期，需重新手動授權 — **無法完全自動化**
- **適合**: 已有 Schwab 帳戶的用戶（但 token 過期是硬傷）

### 11. Tradier
- **URL**: https://tradier.com/
- **費用**: $10/月券商方案即含 SIP 即時
- **即時數據**: ✅ 完整 SIP
- **數據類型**: 報價、OHLCV、選擇權
- **傳輸方式**: REST + WebSocket
- **認證**: Bearer Token（不過期）
- **適合**: 低成本 SIP 即時 + 下單一體化

### 12. Nasdaq Data Link (原 Quandl)
- **URL**: https://data.nasdaq.com/
- **免費方案**: 50,000 次/日
- **即時數據**: ❌ 僅 EOD 與經濟數據
- **數據類型**: EOD、宏觀經濟、另類數據
- **傳輸方式**: REST
- **Python 套件**: `pip install nasdaqdatalink`
- **適合**: 研究用途、宏觀數據、另類數據

---

## 推薦路徑

### 🟢 入門學習路徑（本專案當前階段）
1. **Alpaca** — 免費帳戶 + 紙上交易 + IEX 即時行情，一站式入門
2. **Finnhub** — 補充免費即時串流（WebSocket 監控 50 支股票）
3. 搭配 **yfinance** 下載歷史數據做回測

### 🟡 進階路徑
- **Interactive Brokers** — $10/月完整 SIP + Level 2 + 真實下單
- **Polygon.io** — $199/月專業級 tick 數據

### 最便宜的完整 SIP 即時方案
1. **Charles Schwab** — 免費（但 7 天 token 過期問題）
2. **Interactive Brokers** — $10/月（可減免）
3. **Tradier** — $10/月

---

## 重要注意事項
- **yfinance** 違反 Yahoo TOS，生產環境勿用
- **IEX Cloud** 已關閉，勿再使用
- **Schwab** 的 7 天 token 過期讓全自動化不可行
- **IBKR** 需要本地運行 TWS 或 IB Gateway 軟體
- **Alpaca 免費 IEX 數據** 僅覆蓋約 2% 市場成交量，價差可能較大
- 多數平台禁止數據再分發（redistribution）

---

*調研日期：2026-07-11*
*下一步：選定 API 後建立快速 demo 專案*
