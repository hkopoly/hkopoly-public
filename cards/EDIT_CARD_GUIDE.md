# 卡牌編輯指南

歡迎為 HKOPOLY 設計或修改機會卡與社區基金卡！本指南說明卡牌 JSON 的資料結構，以及編輯卡牌時需要注意的事項。

## 內容位置

卡牌文字存放於 `cards/cards.json`。遊戲網站：[hkopoly.com](https://hkopoly.com)。

卡牌**邏輯**（`action`、`target`、`amount` 等）與 **`text`**（卡牌顯示文字）分開處理。

---

## 檔案結構

`cards.json` 包含三個頂層欄位：

| 欄位 | 說明 |
|------|------|
| `spacePlaceholder` | 移動卡文字中的地點佔位符（固定為 `{space}`） |
| `chance` | 機會卡牌組（陣列） |
| `community` | 社區基金卡牌組（陣列） |

每張卡牌至少需要 `text`（顯示文字）及 `action`（執行動作）。其餘欄位視 `action` 類型而定。

---

## 卡牌基本格式

```json
{
  "text": "銀行派息 HK$50。",
  "action": "money",
  "amount": 50
}
```

| 欄位 | 說明 |
|------|------|
| `text` | 抽到卡牌時顯示嘅描述（支援 `{space}` 佔位符，見下方） |
| `action` | 卡牌觸發嘅遊戲邏輯類型 |

---

## 地點佔位符 `{space}`

`spacePlaceholder` 預設為 `{space}`。當卡牌 `action` 為 `move` 且設有 `target` 時，系統會自動將 `{space}` 替換為**當前地圖**棋盤上對應格子的 `name`。

```json
{
  "text": "前進至{space}。如經過起點，領取 HK$200。",
  "action": "move",
  "target": 13
}
```

若棋盤 `id` 13 的名稱為「旺角」，玩家會看到：「前進至旺角。如經過起點，領取 HK$200。」

> **注意：** `target` 必須對應地圖 `board` 陣列中有效的格子 `id`（0–39）。不同地圖的格子名稱不同，但 `id` 位置一致。

---

## 動作類型（`action`）

### `move`：移動至指定格子

將玩家直接移動到 `target` 指定的棋盤 `id`，並觸發該格子的落點效果（買地、交租等）。

| 欄位 | 必填 | 說明 |
|------|------|------|
| `target` | 是 | 目標格子 `id`（0–39） |
| `collectGo` | 否 | 是否計算經過起點獎金。預設 `true`；設為 `false` 則不發放 |

經過起點獎金取自地圖 `config.GO_SALARY`（預設 HK$200）。若最終停在起點（`id` 0），另加 `config.GO_LANDING_BONUS`（預設 HK$100）。

```json
{ "text": "前進至{space}，領取 HK$300。", "action": "move", "target": 0, "collectGo": true }
```

---

### `money`：收取或繳付現金

| 欄位 | 必填 | 說明 |
|------|------|------|
| `amount` | 是 | 正數為收入，負數為支出 |

支出時若資金不足，會觸發抵押 / 變賣流程；仍不足則可能破產。

```json
{ "text": "銀行派息 HK$50。", "action": "money", "amount": 50 }
{ "text": "繳付貧民稅 HK$15。", "action": "money", "amount": -15 }
```

---

### `jailCard`：出獄許可證

玩家獲得一張出獄許可證，可於坐監時使用或與其他玩家交易。

```json
{ "text": "出獄許可證。", "action": "jailCard" }
```

---

### `jail`：入獄

玩家即時入獄，**不經過起點**，亦**不**領取經過獎金。回合隨即結束。

```json
{ "text": "入獄，不得經過起點。", "action": "jail" }
```

---

### `back`：後退

玩家向後移動指定格數，並觸發落點效果。

| 欄位 | 必填 | 說明 |
|------|------|------|
| `spaces` | 是 | 後退格數（正整數） |

```json
{ "text": "後退三格。", "action": "back", "spaces": 3 }
```

---

### `nearestRailroad`：前進至最近鐵路

將玩家移動至**前方最近**的鐵路格子（`group` 為 `railroad` 的格子）。若前方已無鐵路，則移至棋盤上第一條鐵路（`id` 最小）。

| 欄位 | 必填 | 說明 |
|------|------|------|
| `payDouble` | 否 | 設為 `true` 時，若該鐵路有人擁有，租金加倍 |

```json
{ "text": "前進至最近的港鐵線。租金加倍。", "action": "nearestRailroad", "payDouble": true }
{ "text": "前進至最近的港鐵線。", "action": "nearestRailroad" }
```

---

### `repairs`：房屋維修費

按玩家名下物業的建築數量計算費用並強制繳付。

| 欄位 | 必填 | 說明 |
|------|------|------|
| `house` | 是 | 每間房屋（1–4 幢）的費用 |
| `hotel` | 是 | 每間酒店（5 幢）的費用 |

只計算玩家**擁有**的物業；空地、已抵押物業不計入。

```json
{
  "text": "一般維修：每間房屋 HK$25，每間酒店 HK$100。",
  "action": "repairs",
  "house": 25,
  "hotel": 100
}
```

---

### `payEach`：向每位玩家付款

抽卡玩家向**每一位**其他仍在場的玩家支付 `amount`。資金不足時會逐一向每位玩家結算，可能觸發抵押流程。

```json
{ "text": "你當選主席，向每位玩家支付 HK$50。", "action": "payEach", "amount": 50 }
```

---

### `collectEach`：向每位玩家收款

每一位其他仍在場的玩家向抽卡玩家支付 `amount`。

```json
{ "text": "今日是你生日，向每位玩家收取 HK$10。", "action": "collectEach", "amount": 10 }
```

---

## 兩副牌的分工

| 牌組 | 欄位 | 棋盤觸發 |
|------|------|----------|
| 機會卡 | `chance` | 停在 `type: "chance"` 的格子 |
| 社區基金 | `community` | 停在 `type: "community"` 的格子 |

兩副牌**獨立洗牌**，各自維護棄牌堆；抽完後會重新洗牌。

建議每副牌維持 **16 張**（與現有版本一致），但數量可彈性調整。

---

## 與地圖的關係

卡牌定義與地圖定義**分開存放**：

- 卡牌文字與邏輯 → `cards/cards.json`
- 棋盤格子與規則 → `maps/<map-id>.json`

以下項目取自**地圖** `config`，卡牌本身不會覆寫：

| 設定 | 影響的卡牌行為 |
|------|----------------|
| `GO_SALARY` | `move` 經過起點獎金 |
| `GO_LANDING_BONUS` | 停在起點（`id` 0）的額外獎金 |
| `JAIL_POSITION` | `jail` 入獄目標格子 |
| 鐵路格子位置 | `nearestRailroad` 的移動目標 |

修改 `move` 卡的 `target` 時，請參考 [`maps/EDIT_MAP_GUIDE.md`](../maps/EDIT_MAP_GUIDE.md) 中的棋盤佈局，確保 `id` 指向預期的格子類型。

---

## 設計檢查清單

- [ ] 已更新 `cards/cards.json`
- [ ] 每張卡都有 `text` 和 `action`
- [ ] `action` 值為支援的類型之一（見上方列表）
- [ ] `move` 卡的 `target` 在 0–39 範圍內，且指向合理的格子
- [ ] `money` / `payEach` / `collectEach` 設有 `amount`
- [ ] `back` 設有 `spaces`
- [ ] `repairs` 同時設有 `house` 和 `hotel`
- [ ] 卡牌文字中的金額描述與 `amount` / 費率欄位一致
- [ ] 邏輯無意改動行為時，只改 `text` 即可
- [ ] JSON 格式正確

---

## 提交修改

1. 編輯 `cards/cards.json`
2. 參考本指南確認欄位與動作類型正確
3. 開啟 Pull Request 至本 repo，並簡述修改內容（例如本地化主題、平衡調整、新增卡牌等）

歡迎提交更具地方特色的機會卡與社區基金文案！
