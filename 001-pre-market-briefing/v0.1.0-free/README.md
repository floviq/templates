# floviq · 台股盤前情報推播（Free 版）

> 每個交易日早上 8:00（Asia/Taipei），自動把昨日法人買賣超 / 漲跌停 / 財經新聞整理成一則 AI 摘要訊息，推到你的 Telegram。30 秒讀完，通勤滑完就上班。

版本：**v0.1.0-free** ・ 釋出日期：2026-04-29

---

## 這是什麼

floviq 盤前情報推播是一個 n8n workflow 模板。匯入後設定兩個 credential（OpenAI、Telegram Bot），workflow 會：

1. 抓 TWSE 三大法人買賣超日報
2. 抓 TWSE 全市場每日行情（用來判定漲跌停）
3. 抓鉅亨網財經新聞頭條
4. 餵給 OpenAI（gpt-4o-mini）整理成結構化晨間摘要
5. 拼成純文字訊息推到你指定的 Telegram chat

**適合誰**

- 上班族散戶 — 9:00 開盤但 9:00 通常有會議，沒時間掃盤前消息
- 想被動接收市場概況、不想自己一個一個網站翻的人
- 用 n8n 自動化日常工具的玩家

**輸出長相**（範例 — 每天內容會依當日資料變化）

```
📊 2026-04-29 (週三) 外資延續觀望，台股震盪整理

【外資動向】
外資昨日轉手調節權值股，台積電（2330）出現約 12,300 千股賣壓
中型股部分對聯發科（2454）小幅敲進約 1,800 千股

【漲跌停焦點】
漲停 3 檔（電子零組件為主），跌停 1 檔
焦點落在面板族群，群創（3481）強勢漲停

【新聞重點】
1. Fed 會議釋出鴿派訊號，美股科技股夜盤走強
2. 台積電法說會聚焦先進製程資本支出展望
3. 政院通過半導體研發補助方案

—
※ 本資訊僅供參考，不構成投資建議，投資請自行評估風險
資料來源：TWSE / 鉅亨網
```

---

## 你會收到什麼

- 每交易日早上 **08:00 (Asia/Taipei)** 一則 Telegram 訊息（純文字 + emoji，不含圖）
- 訊息結構固定 4 段：headline / 外資動向 / 漲跌停焦點 / 新聞重點
- 結尾固定帶風險聲明與資料來源
- 平均訊息長度 250~280 中文字，30 秒內讀完

---

## 你需要什麼

| 項目 | 說明 |
|---|---|
| n8n 1.40+ | self-host（Docker / Desktop）或 n8n cloud 都可以 |
| OpenAI API key | platform.openai.com 申請，帳戶儲值 ≥ $5 USD |
| Telegram Bot token | 對 @BotFather 送 `/newbot` 取得，3 分鐘搞定 |
| Telegram chat ID | 你要被推播的 chat（私訊或群組均可） |

---

## 開始使用（30 秒安裝）

### 步驟 1：複製 workflow URL

複製這條 URL：

```
https://raw.githubusercontent.com/floviq/templates/main/001-pre-market-briefing/v0.1.0-free/workflow-free.json
```

### 步驟 2：取得必要 credential

兩個 credential 在 import 後填入即可，現在先準備好。

#### OpenAI API key

1. 到 `https://platform.openai.com/api-keys` 建一把 key
2. 帳戶儲值至少 $5 USD；建議在 `Settings → Limits` 設月度 $1 USD 上限防呆

> 成本估算：gpt-4o-mini 每次執行 ≈ $0.001 USD，每月 22 個交易日 ≈ NT$ 0.7

#### Telegram Bot token + chat ID

1. Telegram 搜 `@BotFather` → 送 `/newbot` → 取個 bot 名字 → 拿到形如 `123456:ABC-...` 的 token
2. 對 bot 送任意訊息（私訊使用），或把 bot 加入群組並設為管理員後在群裡發一句（群組使用）
3. 瀏覽器訪問 `https://api.telegram.org/bot<你的TOKEN>/getUpdates`，找 `chat.id`
   - 私訊：正整數（例 `123456789`）
   - 群組：負整數（含負號，例 `-1001234567890`）

### 步驟 3：n8n Import from URL

1. 打開 n8n → Workflows → 右上 `⋯` → **Import from URL**
2. 貼上步驟 1 的 URL → Save

n8n 會直接從 URL 拉取最新版 workflow，不用下載檔案。

### 步驟 4：選 credential + 設 Config

Import 後先連 credential：

- **OpenAI 摘要** node → Credential 欄選 OpenAI
- **Telegram** node → Credential 欄選 Telegram Bot

接著點 **Config** node 填欄位：

| 欄位 | 必填 | 說明 |
|---|---|---|
| `telegramChatId` | ✅ | 步驟 2 拿到的 chat ID（私訊正整數 / 群組含負號負整數） |
| `topN` | — | 每段顯示前幾名（預設 5；範圍 3~10） |
| `watchlist` | — | 自選股代號清單，逗號分隔（例 `2330,2317,2454`）。Free 版預留欄位，Paid 版啟用 highlight 功能 |
| `promptStyle` | — | AI 摘要語氣：`保守` / `中性` / `活潑`（預設 `中性`） |

### 步驟 5：Execute & 啟用排程

1. 右上 **Execute Workflow** 試跑一次
2. 沒意外幾秒後 Telegram 會收到一則訊息
3. 確認訊息正常 → 點左上 workflow 名稱旁的 **Active** toggle 改 `On`
4. 之後每交易日早上 08:00 自動跑

> ⚠️ 假日 / 盤前資料未產出時 workflow 不會 crash，只會顯示「無資料」字樣。預期行為，等下一個交易日即可。

> 💾 **想要 offline 快照？** 可下載 [v0.1.0-free.zip](./floviq-pre-market-briefing-v0.1.0-free.zip)（[sha256](./floviq-pre-market-briefing-v0.1.0-free.zip.sha256)）作為備份。**不是主路徑**，URL 直連最方便。

---

## 客製化選項

### `promptStyle` 三檔語氣對比

| 設定 | 風格 | 適合 |
|---|---|---|
| `保守` | 用詞最謹慎，多用「觀察」「留意」等中性詞，去除任何情緒詞 | 機構從業者、合規敏感的朋友圈 |
| `中性`（預設） | 標準晨間財經主播語氣，平衡描述 | 一般散戶 |
| `活潑` | 節奏稍快但仍中性，可帶「焦點落在...」「市場目光集中」等開場 | 喜歡讀感稍強的訊息 |

切換方法：改 Config 的 `promptStyle` → Save → 下次 execute 即生效。

### `topN` 調整

預設顯示前 5 名法人買賣超 / 漲跌停。可改 3（精簡）或 10（詳細）。範圍 3~10 之間皆合理。

### `watchlist` 自選股欄位

Free 版此欄位為**預留接口** — 你填進去資料不會影響輸出。Paid 版會啟用：你的自選股若出現在 top 法人或漲跌停，會在訊息加一段「自選股提醒」段落。

---

## 常見問題

### Q1. 為什麼買賣超數字單位是「千股」而不是金額（億）？

TWSE 三大法人日報原始資料即為**股數**（不是金額）。要換算成金額需要乘以「當日均價」，但官方資料沒附均價，估算誤差大且選哪個價（收盤 / 均價 / VWAP）都有爭議。

設計上保留股數方向 + 名稱已足夠傳達訊號（誰被外資掃 / 誰被殺）。AI 摘要會用「規模感」語意（如「大舉敲進」「轉手調節」）補足直觀感受，不直接寫億元金額。

### Q2. 假日或盤前資料還沒出，訊息會怎樣？

workflow 不會 crash。對應段落會顯示「無資料」或「資料暫缺」。等下一個交易日 TWSE 資料更新後，就會恢復正常。

如果你不想在假日收到訊息，可以調整 Schedule node 的 cron expression，或乾脆在週末把 workflow 改 inactive。

### Q3. AI 訊息出現「應該買」「目標價」「會漲」等字眼怎麼辦？

這是設計上的紅線違規 — system prompt 已加多重防線禁止這類用語。**如果真的出現**，請：

1. **立刻停用 workflow**（toggle Active off）
2. 把該則訊息截圖
3. 寄到 `hello@floviq.com`

我們會把 case 加入合規測試集，並在最近一版修補。

### Q4. OpenAI 帳戶要儲值多少？

最低 $5 USD 起跳（OpenAI 規定）。

實際每月成本：每交易日 1 次 × 22 日 × $0.001 ≈ **$0.022 USD**（約 NT$ 0.7）。儲 $5 大概可以用 2 年。

建議在 OpenAI dashboard 設「Monthly limit」$1 USD 防呆，不會被意外刷爆。

### Q5. 如何把訊息發到群組而不是私訊？

兩個關鍵差異：

1. **加 bot 進群組**並設為管理員（私訊則不需要）
2. **chat ID 是負整數**（含負號），私訊則是正整數

把含負號的整段填進 Config 的 `telegramChatId` 即可。

### Q6. 我可以同時推給私訊跟群組嗎？

Free 版單一 chat。如果需要多目標推播（私訊 + 群組 + LINE + Discord），請考慮 Paid 版的多通路加值。

### Q7. 我可以改 AI 的輸出格式嗎？

Free 版開放：`promptStyle` 三檔語氣切換 + `topN` 調整。

不開放：直接編輯 system prompt（會破壞合規防線與訊息結構）。Paid 版會開放完整 prompt 客製。

### Q8. 我可以改用其他 OpenAI 模型嗎（gpt-4o / o1）？

技術上可以 — 在 OpenAI 摘要 node 改 modelId。但要注意：

- gpt-4o 比 gpt-4o-mini 貴約 30 倍
- o1 系列不支援 `response_format: json_object`，會破壞 workflow

預設 gpt-4o-mini 對結構化任務已經夠用，改之前先想清楚。

---

## Free 版限制

- **單一推播通路**：Telegram（無 LINE / Discord）
- **自選股 highlight 功能**：未啟用（欄位保留）
- **AI prompt 客製**：限制在 `promptStyle` 三檔 enum
- **產業關鍵字過濾 / 警示閾值**：未開放
- **設定協助**：自助為主，社群 Q&A 為輔

---

## 升級到 Paid 版

Free 版給你預設好的盤前情報；Paid 版則是 floviq 幫你把這支 workflow 調整成符合你的看盤習慣 — 你的通路、你的自選股、你的訊息語氣、你的關注產業。

Paid 版加值：

| 加值項目 | 內容 |
|---|---|
| 多通路 | LINE Flex Message（視覺卡片）/ Discord webhook |
| 個人化 | 自選股 highlight（自選股出現在 top 時加段提醒） |
| AI 深度客製 | 整段 system prompt override / 自訂 sections / 警示閾值 |
| 產業關鍵字 | 半導體 / AI / 金融 / 電動車 等過濾條件 |
| 設定協助 | Email 諮詢 + 一次客製設定協助（含 LINE OA / Discord 串接） |
| Bug SLA | 7 天回應 |

**取得 Paid 版**：Paid 版預計 2026-Q2 上架，屆時將同步更新本檔的取得連結。屆時可至 `https://floviq.tw` 查看最新資訊。

---

## 免責聲明

完整條款見 `LICENSE.txt`。要點：

- 本工具為**資訊整理與通知工具**，不構成任何形式的投資建議、財務建議或交易訊號
- 一切投資決定與其結果由使用者自行承擔，floviq 不負任何投資損益責任
- 外部資料（TWSE / 鉅亨網）正確性由原始資料提供者負責
- 「現況」（AS-IS）提供，不附帶適售性或特定用途適用性保證

---

## 文件 / 連絡

- 授權問題：`hello@floviq.com`
- 版本：v0.1.0-free
- 釋出日期：2026-04-29
- 客戶服務語言：繁體中文（zh-TW）為主，英文（en）為輔
