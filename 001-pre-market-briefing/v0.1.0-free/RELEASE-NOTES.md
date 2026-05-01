# floviq · 001-pre-market-briefing · v0.1.0-free

Released: 2026-04-29
Last patched: 2026-05-01

這是 floviq 第一個對外發行的 n8n template，採 Free 版定位（lead magnet）。

---

## Patches (in-version updates · 不 bump 版本)

純文字 / metadata 修正會直接 patch 到既有版本目錄，不另發新版本（無行為變化）。Workflow JSON 內 node 邏輯與 schema 變動才會 bump。

### 2026-05-01 · cost numbers + email patch

- **`hello@floviq.com` → `support@floviq.tw`** 跨 LICENSE / README / RELEASE-NOTES — 舊 email 從未啟用會 bounce，已切到實際 inbox
- **OpenAI 成本估算對齊 tiktoken 精確算** 跨 README / RELEASE-NOTES / `prompts/summary-prompt.md` / `workflow-free.json` sticky + node notes：
  - 舊（char heuristic 估算）：每次執行 `~$0.001 USD`、月成本 `~$0.022 USD`、`NT$ 0.7`
  - 新（tiktoken 對 cl100k_base 精確算）：每次執行 `~$0.0004 USD`、月成本 `~$0.009 USD`、`NT$ 0.3`
  - 根因：cl100k_base 對 zh-TW + JSON 實測 0.4–0.6 tokens/char，舊估算用 en-only 1.3 ratio 高估約 2×。**實際成本只低不高**，不影響你的 OpenAI 帳戶儲值規劃（建議的 $5 USD 仍夠用 1 年以上）。
- 重新打包 distribution zip + 更新 sha256

未動：`workflow-free.json` 任何 node 邏輯 / schema / 排程 / 推播格式。已導入並啟用此 workflow 的客戶 **無需任何動作**，現有部署繼續正常運作。

---

## Highlights

- **三資料源 aggregator**
  - TWSE 三大法人買賣超日報（T86）
  - TWSE 全市場每日行情（MI_INDEX，用來判定漲跌停 + 大盤指數）
  - 鉅亨網（CNYES）財經頭條 Top 3
- **AI 摘要**：gpt-4o-mini，結構化 JSON 輸出，固定 4 段（headline / 外資動向 / 漲跌停 / 新聞）
- **Telegram 純文字推播**：避開 MarkdownV2 escape 風險（台股代號常含括號）
- **客戶可調設定**：`telegramChatId` / `topN` / `watchlist` 預留 / `promptStyle` 三檔（保守 / 中性 / 活潑）
- **n8n 原生 node**：Schedule / OpenAI / Telegram 全用原生 node，credential 走 n8n credential UI（不放 .env）

---

## Cost Estimate

- OpenAI gpt-4o-mini：≈ $0.0004 USD / 次執行（tiktoken 精確算）
- 月費（每交易日 1 次 × 22 交易日）：**≈ $0.009 USD**（約 NT$ 0.3）
- OpenAI 帳戶最低儲值 $5 USD（可用 1 年以上）

---

## Compatibility

- **n8n**：1.40+
- **環境**：self-host（Docker / Desktop / VPS）與 n8n cloud 兩者皆已驗證可用
- **時區**：Asia/Taipei（cron 排程預設週一至週五 08:00）
- **語言**：訊息輸出為繁體中文（zh-TW）

---

## Known Limitations (Free)

- **單一推播通路**：Telegram（無 LINE Flex / Discord webhook）
- **watchlist 為預留欄位**：可填入但不影響輸出（Paid 版啟用 highlight）
- **AI prompt 客製**：限制在 `promptStyle` 三檔 enum，不開放編輯 system prompt
- **產業關鍵字過濾 / 警示閾值**：未開放
- **多 chat 推播**：單一 chat ID（私訊或群組擇一）

---

## Compliance

- 純資訊整理工具定位，不執行交易、不提供投資建議
- AI system prompt 內建合規防線：禁用「買進 / 賣出 / 應該買 / 目標價 / 會漲」等明確投資建議用語
- 訊息結尾固定帶風險聲明：「本資訊僅供參考，不構成投資建議，投資請自行評估風險」
- 完整授權與免責條款見 `LICENSE.txt`

---

## File Manifest

```
floviq-pre-market-briefing/
├── README.md              ← 客戶安裝與使用指南
├── RELEASE-NOTES.md       ← 本檔
├── LICENSE.txt            ← 個人使用授權 + 免責
├── workflow-free.json     ← n8n workflow 主檔（10 nodes）
└── prompts/
    └── summary-prompt.md  ← AI system prompt 副本（供你閱讀 / 客製化參考；實際 system prompt 已嵌入 workflow-free.json 的 OpenAI node 內，修改本檔**不會**影響 workflow 執行）
```
