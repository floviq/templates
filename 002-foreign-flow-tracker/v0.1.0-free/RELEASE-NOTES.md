# floviq · 002-foreign-flow-tracker · v0.1.0-free

Released: 2026-05-14

Initial free release. n8n workflow template — 每週一自動聚合 TWSE T86 外資進出數據，推送到 Telegram。

---

## v0.1.0-free · 2026-05-14 — initial release

### 功能

- 每週一 08:00 (Asia/Taipei) 定時觸發，抓上週 5 個交易日的 TWSE T86 三大法人買賣超日報
- 自動計算有效交易日（過濾 `holidaySkipList` 假日 + 跳過 TWSE API 回傳「無資料」的日期）
- 外資欄位：T86「外陸資（境外法人 + 陸資），不含外資自營商部位」（idx 4 欄位，單位：股）
- 4 個切片計算：週淨買超 Top N / 週淨賣超 Top N / 連續買超 ≥ N 日 / 連續賣超 ≥ N 日
- 顯示單位換算：股 → 千張（1 張 = 1000 股，週淨額 / 1000 / 1000）
- 整週皆假日（空週）fallback：推送「上週無交易日資料（連假休市）」通知，不空白
- 純文字推送至 Telegram（無 parse_mode，避免台股名稱含括號的 escape 問題）
- `retryOnFail: true, maxTries: 2, waitBetweenTries: 3000` 於 HTTP T86 + Telegram node

### Workflow 結構

7 functional nodes（線性串接）+ 4 sticky notes：

```
Schedule → Config → Calc Last Week Days → HTTP T86 → Aggregate → Format → Telegram
```

- **Schedule**：cron `0 0 8 * * 1`（每週一 08:00，Asia/Taipei）
- **Config**：Set node v3.4，4 個 customer config keys
- **Calc Last Week Days**：Code node — 計算上週一~五有效交易日清單，過濾假日
- **HTTP T86**：httpRequest v4.2 — 逐日抓 TWSE T86（n8n per-item 自動 loop，共 5 req）
- **Aggregate**：Code node — 聚合 5 天，計算 4 切片，consecutive sorted + sliced to topN
- **Format**：Code node — 千張換算 + disclaimer footer 強制 append
- **Telegram**：native Telegram node v1.2 — sendMessage

Sticky notes 4 張（整體說明 / Config 設定 / TWSE T86 資料 / Telegram 推播）：純 customer-facing 安裝說明，不含內部標記。

### Customer config（Config node 4 keys）

| 欄位 | 預設 | 說明 |
|---|---|---|
| `telegramChatId` | `REPLACE_ON_IMPORT` | 必填。Telegram chat ID |
| `topN` | `5` | 淨買超 / 淨賣超顯示前幾名 |
| `consecutiveThreshold` | `3` | 連續同向天數門檻 |
| `holidaySkipList` | 2026 全年 + 2027/1/1 | 逗號分隔 YYYYMMDD 字串 |

### Credentials

- **Telegram Bot**：n8n Credential（workflow 匯入後在 n8n UI 點選一次連結）
- **OpenAI**：不需要（本版本無 AI 摘要）

### 合規

- 0 個股建議 / 預測 / 「值得關注」字眼
- 訊息 footer 固定附「外資」定義說明（外陸資含陸資、不含外資自營商）
- 訊息結尾固定帶風險聲明：「本資訊僅供參考,不構成投資建議,投資請自行評估風險」
- workflow `_meta` / sticky notes 全 customer-clean（0 內部路徑 / spec 標記）

---

## 下一步規劃中

以下功能處於規劃中，不附期程與承諾：

- **AI 週報摘要**：把 4 切片數據透過 GPT-4o-mini 生成白話週報
- **多通路推播**：LINE Flex Message / Discord Webhook（Paid 版）
- **自選股 highlight**：自選股若出現在任一切片，加一段「自選股本週動態」段落（Paid 版）
- **Error handler**：失敗通知 + 自動 retry 流程

---

## File Manifest

```
002-foreign-flow-tracker-v0.1.0-free/
├── README.md              ← 客戶安裝與使用指南（本次釋出）
├── RELEASE-NOTES.md       ← 本檔
├── LICENSE.txt
└── workflow-free.json     ← n8n workflow 主檔（v0.1.0，11 nodes）
```
