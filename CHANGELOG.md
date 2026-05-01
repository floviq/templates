# floviq/templates · 變動紀錄

跨所有 template、跨所有版本的時間軸。每次 push 動到 customer-facing 內容都登記一筆。

**版本規則**：

- 新版本 = 新 folder（譬如 `001-pre-market-briefing/v1.0.0-paid/`）。不覆蓋舊版本目錄
- 同版本內**純文字 / metadata 修正**（cost 數字、email、typo、錯字）= patch，直接改既有目錄並重產 zip + sha256，不另開 folder
- Workflow JSON 內 node 邏輯 / schema / 排程 / 推播格式變動 = bump 版本，新 folder

每筆變動格式：日期 · `template/version` · 變動類型 · 摘要

---

## 2026-05-01

- `001-pre-market-briefing/v0.1.0-free` · **patch** · cost numbers 對齊 tiktoken 精確算（舊估算高估約 2×，實際成本只低不高）+ 客服 email `hello@floviq.com` → `support@floviq.tw`（舊 email 從未啟用會 bounce）+ 重產 distribution zip + 更新 sha256。**未動**任何 workflow 邏輯，已部署客戶無需動作。詳見 `001-pre-market-briefing/v0.1.0-free/RELEASE-NOTES.md` 內 Patches 段。

## 2026-04-29

- `001-pre-market-briefing/v0.1.0-free` · **initial release** · floviq 第一個對外發行的 n8n template。三資料源 aggregator（TWSE 三大法人 / TWSE 全市場行情 / 鉅亨網新聞）+ gpt-4o-mini AI 摘要 + Telegram 推播。詳見 `001-pre-market-briefing/v0.1.0-free/RELEASE-NOTES.md`。

## 2026-04-28

- repo init · 建立 floviq/templates 公開分發 mirror。
