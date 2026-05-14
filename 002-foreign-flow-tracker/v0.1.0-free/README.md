<p align="center">
  <img src="./assets/floviq-logo-no-bg.png" alt="floviq" width="120" />
</p>

# floviq · 外資進出週報（Free 版）

> 每週一早上 8:00（Asia/Taipei），自動把上週 5 個交易日的外資進出資料聚合成 4 個切片，推到你的 Telegram。週一上班前 3 分鐘讀完，知道外資上週買了誰、賣了誰。

版本：**v0.1.0-free** ・ 釋出日期：2026-05-14

---

## 這是什麼

floviq 外資進出週報是一個 n8n workflow 模板。匯入後設定一個 credential（Telegram Bot），workflow 會在每週一自動：

1. 計算上週一到上週五的交易日清單（自動跳國定假日）
2. 逐日抓 TWSE T86 三大法人買賣超日報（外資欄位）
3. 聚合 5 個交易日數據，計算 4 個切片
4. 格式化成純文字推送到你的 Telegram

**4 個切片**：

- 外資週淨買超前 N 名（上週累計正向最大）
- 外資週淨賣超前 N 名（上週累計負向最大）
- 連續買超 ≥ N 日（每一個有效交易日都淨買）
- 連續賣超 ≥ N 日（每一個有效交易日都淨賣）

**適合誰**

- 上班族散戶 — 平日沒時間每天看外資動向，想在週一規劃當週策略前有個數據概覽
- 不想每天盯盤、偏好週節奏的投資人
- 已在跑「floviq 盤前情報推播」、想補一個週維度視角的人
- 習慣 n8n 自動化工具的玩家

**輸出長相**（格式示意 — 每週實際數值依當週 TWSE 資料變化）

```
📅 YYYY/MM/DD ~ YYYY/MM/DD 外資進出週報

🔼 外資淨買超 Top 5（上週累計）
1. [個股名稱] ([代號])   +[XXX] 千張
2. [個股名稱] ([代號])   +[XX] 千張
3. [個股名稱] ([代號])   +[XX] 千張
4. [個股名稱] ([代號])   +[XX] 千張
5. [個股名稱] ([代號])   +[X] 千張

🔽 外資淨賣超 Top 5（上週累計）
1. [個股名稱] ([代號])   [−XXX] 千張
2. [個股名稱] ([代號])   [−XX] 千張
3. [個股名稱] ([代號])   [−XX] 千張
4. [個股名稱] ([代號])   [−XX] 千張
5. [個股名稱] ([代號])   [−X] 千張

🟢 連續買超 ≥ 3 日
- [個股名稱] ([代號])   [N] 日連續
- [個股名稱] ([代號])   [N] 日連續
- (本週無)

🔴 連續賣超 ≥ 3 日
- [個股名稱] ([代號])   [N] 日連續
- (本週無)

📊 資料區間: YYYY/MM/DD ~ YYYY/MM/DD · 5 個交易日
📍 資料來源: 證交所 T86 三大法人買賣超日報
※「外資」= 外陸資（境外法人 + 陸資）· 不含外資自營商部位

※ 本資訊僅供參考,不構成投資建議,投資請自行評估風險
```

---

## 你會收到什麼

- 每週一早上 **08:00 (Asia/Taipei)** 一則 Telegram 訊息（純文字 + emoji，不含圖）
- 推播節奏：**每週一次**（週節奏，不同於「floviq 盤前情報推播」的每日推播）
- 訊息結構固定 4 段：週淨買超排名 / 週淨賣超排名 / 連續買超清單 / 連續賣超清單
- 結尾固定帶「外資」定義說明 + 風險聲明 + 資料來源
- 平均訊息長度約 500-800 字元，3 分鐘內讀完
- 若整週連假（無交易日），推送「上週無交易日資料（連假休市）」一行通知，不空白
- 顯示單位：**千張**（1 張 = 1000 股，週淨額 / 1000 / 1000 = 千張）

---

## 你需要什麼

| 項目 | 說明 |
|---|---|
| n8n 1.40+ | self-host（Docker / Desktop）或 n8n Cloud 都可以 |
| Telegram Bot token | 對 @BotFather 送 `/newbot` 取得，5 分鐘搞定 |
| Telegram chat ID | 你要被推播的 chat（私訊或群組均可） |

**不需要**：OpenAI API key（本版本無 AI 摘要，純數據聚合）、付費資料源（TWSE T86 是公開資料）

---

## 開始使用（5 步）

### 1. 環境準備

n8n 1.40+ 兩種環境都可以跑，選一個：

- **self-host**（推薦給本來就熟 Docker 的人）— Docker / n8n Desktop / VPS
- **n8n Cloud** — n8n.io 註冊帳號

本 workflow 在兩個環境都已驗證可用，設定欄位名稱相同，差別只在 credential 設定位置。

### 2. 取得必要 credential

#### Telegram Bot token

1. Telegram 搜尋 `@BotFather` → 對話框送 `/newbot`
2. 照指示取個 bot 名字（username 必須以 `bot` 結尾，例如 `my_weekly_tracker_bot`）
3. 拿到一串形如 `123456789:ABCdef-...` 的 token，**這就是 bot token**

#### Telegram chat ID

**私訊**（個人使用）：

1. 在 Telegram 搜尋你剛建的 bot 名字 → 點開 → 對它送任意一句話（例：`hi`）
2. 瀏覽器訪問 `https://api.telegram.org/bot<你的TOKEN>/getUpdates`
3. JSON 中找 `result[0].message.chat.id`，是一個正整數（例：`123456789`）

**群組**（要分享給家人 / 朋友圈）：

1. 把 bot 加入群組並設為管理員
2. 群組裡傳任意一句話
3. 同上訪問 `getUpdates`，找 `chat.id`，**是負整數**（含負號，例：`-1001234567890`）
4. 整段（含負號）填到 Config 的 `telegramChatId`

### 3. 在 n8n 設定 Telegram credential

1. 開 n8n → 左側 **Credentials** → 右上 **+ Add credential**
2. 搜尋 `Telegram` → 選 **Telegram Bot**
3. 把 token 貼進 Access Token 欄 → **Save**

### 4. Import workflow + 連 credential

1. n8n → 左側 Workflows → 右上 `+` → **Import from File**
2. 選 `workflow-free.json`
3. workflow 開啟後，點 **Telegram** node → Credentials 欄選剛建的 Telegram Bot credential
4. 點 **Config** node → 把 `telegramChatId` 欄位的 `REPLACE_ON_IMPORT` 換成你的 chat ID

### 5. 試跑 + 啟用排程

1. 右上 **Execute Workflow** 手動跑一次（不必等週一）
2. 幾秒後 Telegram 應收到一則訊息（抓的是「上週」資料）
3. 確認格式正確、disclaimer 在 → 點左上 workflow 名稱旁邊的 **Active** toggle 改 `On`
4. 之後每週一早上 08:00 自動推播

> 第一次 Execute 請確認 Telegram credential 已連好，否則 Telegram node 會紅燈報 authentication error。

---

## 客製化選項

| 欄位 | 預設 | 說明 |
|---|---|---|
| `telegramChatId` | `REPLACE_ON_IMPORT` | 必填。Telegram chat ID（私訊用正整數，群組用含負號的負整數） |
| `topN` | `5` | 每段顯示前幾名。合理範圍 3~10 |
| `consecutiveThreshold` | `3` | 連續幾天同向才進「連續買超 / 賣超」清單。預設 3 天，想更嚴格可改 4 或 5 |
| `holidaySkipList` | 2026 全年國定假日 | 逗號分隔 YYYYMMDD 字串。每年年底自己加新一年的清單 |

**改設定方法**：點 Config node → 修改對應欄位值 → Save → 下次 Execute 即生效。不需要重新 import。

**`consecutiveThreshold` 說明**：設為 3 代表「上週 5 個交易日中，每一個有效交易日都持續買（或賣）超的個股才進清單」。這是全 valid 天數的條件，不是「任意連續 3 天」。

**`holidaySkipList` 維護**：預設清單已含 2026 全年主要國定假日 + 2027/1/1。農曆春節與颱風日不在此清單內，因為 TWSE 那天若未開盤，T86 API 會回傳「無資料」，程式自動跳過。每年年底建議把下一年的清單補進來（只需要改 Config node，不動 workflow）。

**進階客製規劃中**：AI 週報摘要（自動把 4 切片轉成白話週報）、多通路推播（LINE / Discord）、自選股 highlight，均規劃於 Paid 版。

---

## 常見問題

### Q1. 這個跟「floviq 盤前情報推播」有什麼不同？

| | floviq 盤前情報推播 | floviq 外資進出週報 |
|---|---|---|
| 推播節奏 | 每個交易日 08:00 | 每週一 08:00 |
| 資料維度 | 昨日法人買賣超 + 漲跌停 + 新聞 | 上週 5 天外資買賣超聚合 |
| 適合心智模型 | 每日通勤掃盤前訊號 | 週初回顧外資動向、規劃當週 |
| AI 摘要 | 有（gpt-4o-mini 晨間摘要）| 無（純數據，AI 版規劃中）|
| 需要 OpenAI key | 是 | 否 |

兩個可以同時跑，互補不重複。

### Q2. 為什麼數字單位是「千張」而不是億元？

TWSE T86 原始資料單位是「股」。要換算成金額需要乘以當日均價，但官方資料沒附均價，且選哪個價位（收盤 / 均價 / VWAP）都有爭議。

直接顯示「千張」保留原始信號方向與相對規模，不引入換算誤差。每月 / 每週同單位比較也一致。

### Q3. 假日跑了會怎樣？

workflow 每週一 08:00 固定觸發，包含連假後第一個週一。calc 節點會算出「上週一~上週五」日期，並過濾 `holidaySkipList` 內的假日日期。TWSE 那天若未開盤，T86 API 回傳「無資料」，程式自動跳過該天，最終聚合有效交易日的數據。

若整週都是假日（例如農曆春節長假），推送「上週無交易日資料（連假休市）」通知，workflow 不會 crash。

### Q4. 「外資」的定義是什麼？

本工具的「外資」指 TWSE T86 中的「**外陸資（境外法人 + 陸資），不含外資自營商部位**」。這是台灣證交所公告的標準分類，也是一般財經媒體引用的同一條數字。訊息 footer 固定有一行定義說明。

### Q5. 「連續買超」的條件是怎麼算的？

「連續買超 ≥ N 日」代表：在當週所有**有效交易日**（排除假日 / 休市）中，每一天都維持正向外資淨買超。不是「任意連續 N 天」，而是「整週每個開盤日都買」。有效交易日少於 N 天時（例如：長假週只有 2 天），此清單可能為空，屬正常行為。

### Q6. 可以同時推給私訊和群組嗎？

Free 版單一 Telegram chat 目標。如果需要多目標推播，Paid 版支援多通路（LINE Flex / Discord webhook）。

### Q7. 這週一直沒收到訊息，怎麼排查？

1. n8n → 左側 **Executions** → 找到週一那次執行 → 看哪個 node 紅燈
2. 常見原因：Telegram credential 沒連好（authentication error）、`telegramChatId` 填的是 `REPLACE_ON_IMPORT` 沒改
3. 手動點 **Execute Workflow** 重跑一次，比等下週一快很多

---

## 合規與資料來源

本 workflow 資料唯一來源為**台灣證券交易所（TWSE）三大法人買賣超日報（T86）**。

- 公開資料官網：`https://www.twse.com.tw/zh/trading/fund/T86.html`
- 資料性質：歷史交易事實彙整，TWSE 每交易日 17:30 後公告
- 本工具做的事：抓取 → 數學聚合（加總 / 排序）→ 格式化推送

**本工具不做的事**：

- 不給任何個股建議（「應該買」「建議」「值得關注」「題材」等字眼不出現）
- 不預測股價漲跌
- 不提供交易訊號

> ※ 本資訊僅供參考，不構成投資建議，投資請自行評估風險

---

## 已知限制 / 注意事項

- **連日程序非容錯設計**：若 n8n instance 週一 08:00 時掛掉未執行，不會自動補跑。手動點 Execute 可以補抓上週資料（只要在下週一前執行）。
- **`holidaySkipList` 每年需維護**：颱風 / 補班日不在預設清單內。TWSE 如未開盤，T86 API 自動回傳「無資料」，workflow 能自動跳過；但若颱風日補班開盤，則會正常抓到那天的資料。
- **農曆春節不在預設清單**：農曆新年日期每年不同。2026 農曆年假期間（約 1/26~2/2）的每一天，TWSE 未開盤 → T86 API 自動回「無資料」→ 程式跳過。使用者無需特別設定，但若想讓整週觸發跳過，可在 `holidaySkipList` 加入那段日期。
- **訊息長度**：預設 Top 5 × 4 切片，約 500-800 字元，遠在 Telegram 4096 字元上限內。`topN` 改到 10 大約 1200 字元，仍安全。
- **Free 版無 AI 摘要**：4 切片是純數據輸出，無白話解讀。AI 週報摘要規劃於 Paid 版。

---

## 授權

個人使用授權。完整條款見 `LICENSE.txt`。

要點：

- 可用於個人投資資訊整理
- 可為個人需求修改 workflow
- 不可轉售、不可分享 workflow 檔案給他人
- 不可用於提供商業訊號服務

---

## floviq 系列其他模板

| 模板 | 說明 | 節奏 |
|---|---|---|
| [001 · 台股盤前情報推播](../../001-pre-market-briefing/) | 每交易日早上 8:00 — 三大法人買賣超 + 漲跌停 + AI 晨間摘要 → Telegram | 每日 |
| **002 · 外資進出週報（本模板）** | 每週一早上 8:00 — 外資週淨買賣超 4 切片 → Telegram | 每週 |

---

<p align="center">
  Made by <a href="https://floviq.tw">floviq.tw</a>
</p>
