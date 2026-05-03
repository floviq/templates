# floviq · 001-pre-market-briefing · v0.1.1-free

Released: 2026-05-03

Patch release fixing a date-resolution bug in v0.1.0-free.

---

## What's Fixed

**Bug**：cron 08:00 早上 query「今天」TWSE 三大法人 / 漲跌停 / 大盤資料拿不到（盤後資料當日 17:00 才公告），導致訊息可能空白或缺資料。

**Fix**：新增 `Calc TradingDate` node 自動算「最近一個交易日」(skip 週末 + TWSE T86 stat retry)，週一早上自動拿上週五的盤後資料，連假後第一天自動拿連假前最後一個交易日。

**訊息結構升級**：訊息頭加副標 `資料日期：YYYY-MM-DD（最近交易日盤後）· 新聞：即時頭條`，三類資料時效一目了然。

**AI prompt 補時效 context**：法人 / 漲跌停 / 大盤是「上一交易日盤後」資料；新聞是「即時頭條」。AI 摘要用詞自動區分時效。

---

## Compatibility

- v0.1.0-free 用戶 upgrade 步驟：
  1. import 新版 `workflow-free.json` (新建一個 workflow，**不會 in-place 覆蓋舊的**)
  2. 在新 workflow 的 Config node 把舊版 4 個欄位值（`telegramChatId` / `topN` / `watchlist` / `promptStyle`）對應填回
  3. 兩個 credential（OpenAI / Telegram）重新 link
  4. 停用舊 workflow → 啟用新 workflow
  5. Schema 沒變，是「同位置覆蓋值」不是「重新設計」
- 沒裝過：照 README.md 5 步流程安裝

---

## Cost Estimate

OpenAI gpt-4o-mini：每次執行 ≈ $0.0004 USD，月費（每交易日 1 次 × 22 交易日）≈ $0.009 USD。額外多 1~5 次 TWSE GET（連假時 retry，無感）。

---

## File Manifest

```
floviq-pre-market-briefing/
├── README.md              ← 客戶安裝與使用指南
├── RELEASE-NOTES.md       ← 本檔
├── LICENSE.txt
├── workflow-free.json     ← n8n workflow 主檔（v0.1.1，15 nodes）
└── prompts/
    └── summary-prompt.md  ← AI system prompt 副本
```

---

## Compliance

- 純資訊整理工具定位，不執行交易、不提供投資建議
- AI system prompt 內建合規防線
- 訊息結尾固定帶風險聲明
- 完整授權見 LICENSE.txt
