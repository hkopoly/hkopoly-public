# 地圖編輯指南

歡迎為 HKOPOLY 設計新棋盤！本指南說明地圖 JSON 的資料結構，以及建立或修改地圖時需要注意的事項。

## 內容位置

地圖 JSON 存放於 `maps/`。遊戲網站：[hkopoly.com](https://hkopoly.com)。

參考檔案：`maps/urban1.json`、`maps/rural1.json`。

---

## 檔案結構

每個地圖 JSON 包含七個頂層欄位：

| 欄位 | 說明 |
|------|------|
| `id` | 地圖唯一識別碼（例如 `urban1`） |
| `name` | 顯示名稱（例如 `港九核心`） |
| `config` | 遊戲規則設定（共用數值；見下方） |
| `board` | 40 格棋盤定義（`id` 0–39） |
| `groups` | 地產顏色組定義 |
| `chance` | 機會卡牌組（陣列） |
| `community` | 社區基金卡牌組（陣列） |

棋盤、地產組與卡牌定義存放於**同一個**地圖檔案，方便按地圖主題撰寫本地化卡牌文案。

---

## 棋盤佈局（`id` 0–39 的位置）

棋盤共 **40 格**，玩家按順時針方向移動：`0 → 1 → 2 → … → 39 → 0`。

```
  0 ─── 1 ─── 2 ─── 3 ─── 4 ─── 5 ─── 6 ─── 7 ─── 8 ─── 9 ── 10
  │                                                           │
 39                                                          11
  │                                                           │
 38                                                          12
  │                                                           │
 37                                                          13
  │                                                           │
 36                                                          14
  │                                                           │
 35                        （棋盤中央）                       15
  │                                                           │
 34                                                          16
  │                                                           │
 33                                                          17
  │                                                           │
 32                                                          18
  │                                                           │
 31                                                          19
  │                                                           │
 30 ── 29 ── 28 ── 27 ── 26 ── 25 ── 24 ── 23 ── 22 ── 21 ── 20
```

| 位置 | `id` | 角落 / 邊 | 預設用途 |
|------|------|-----------|----------|
| 左上角 | **0** | 頂行起點 | `go`（起點） |
| 頂行（左→右） | **1–9** | 頂邊 | 地產、機會、稅項等 |
| 右上角 | **10** | 頂行終點 | `jail`（探監） |
| 右邊（上→下） | **11–19** | 右邊 | 地產、社區基金等 |
| 右下角 | **20** | 底行終點 | `parking`（免費泊車） |
| 底行（右→左） | **21–29** | 底邊 | 地產、機會等 |
| 左下角 | **30** | 底行起點 | `gotojail`（即時入獄） |
| 左邊（下→上） | **31–39** | 左邊 | 地產、稅項等 |

> **注意：** `id` 0 在畫面上是**左上角**（起點），不是傳統大富翁的右下角。`id` 10 必須是 `jail`（探監），`id` 30 必須是 `gotojail`（即時入獄）。

---

## 格子類型（`board[].type`）

| `type` | 說明 | 常用欄位 |
|--------|------|----------|
| `go` | 起點 | `name`, `subtitle`（可選，顯示經過/停留獎金） |
| `property` | 地產 | `name`, `price`, `rent`, `group` |
| `railroad` | 鐵路 | `name`, `price`, `rent`, `group: "railroad"` |
| `utility` | 公用事業 | `name`, `price`, `rent`, `group: "utility"` |
| `chance` | 機會卡 | `name` |
| `community` | 社區基金 | `name` |
| `tax` | 稅項 | `name`, `amount` |
| `jail` | 探監（只可探監，不入獄） | `name`, `cellLabel` |
| `parking` | 免費泊車 | `name` |
| `gotojail` | 即時入獄 | `name` |

### 租金陣列 `rent`

地產 `rent` 為長度 6 的陣列，依序代表：

`[空地租金, 1 幢, 2 幢, 3 幢, 4 幢, 酒店]`

鐵路 `rent` 為長度 4 的陣列，依擁有 1–4 條路線遞增。

公用事業 `rent` 為骰子倍率陣列，依擁有間數遞增（例如 `[4, 10]` 表示 1 間 ×4、2 間 ×10；租金 = 移動骰子點數 × 倍率）。

---

## 地產組（`groups`）

每個顏色組需要以下欄位：

```json
"shamShuiPo": {
  "id": "shamShuiPo",
  "name": "深水埗",
  "color": "#04642c",
  "icon": "/icons/hk_18d_ssp.webp",
  "spaceIds": [6, 8, 9],
  "houseCost": 50
}
```

| 欄位 | 說明 |
|------|------|
| `id` | 組別 key，須與 `board[].group` 一致 |
| `name` | 顯示名稱 |
| `color` | 地產色帶顏色（HEX） |
| `icon` | 圖示（見下方） |
| `spaceIds` | 此組所有格子的 `id` 陣列 |
| `houseCost` | 起樓費用（鐵路 / 公用事業可省略） |

每張地圖應有 **8 個地產顏色組**、**1 個鐵路組**（`railroad`）、**1 個公用事業組**（`utility`）。

---

## 圖示（`groups[].icon`）

### 香港 18 區地產圖示（WebP）

地產顏色組可使用內建區徽圖片，路徑格式為 `/icons/hk_18d_<代碼>.webp`：

| 代碼 | 檔名 | 香港行政區 |
|------|------|------------|
| `central` | `/icons/hk_18d_central.webp` | 中西區 |
| `wc` | `/icons/hk_18d_wc.webp` | 灣仔 |
| `east` | `/icons/hk_18d_east.webp` | 東區 |
| `south` | `/icons/hk_18d_south.webp` | 南區 |
| `ytm` | `/icons/hk_18d_ytm.webp` | 油尖旺 |
| `ssp` | `/icons/hk_18d_ssp.webp` | 深水埗 |
| `kc` | `/icons/hk_18d_kc.webp` | 九龍城 |
| `kt` | `/icons/hk_18d_kt.webp` | 觀塘 |
| `wts` | `/icons/hk_18d_wts.webp` | 黃大仙 |
| `north` | `/icons/hk_18d_north.webp` | 北區 |
| `tp` | `/icons/hk_18d_tp.webp` | 大埔 |
| `st` | `/icons/hk_18d_st.webp` | 沙田 |
| `sk` | `/icons/hk_18d_sk.webp` | 西貢 |
| `tm` | `/icons/hk_18d_tm.webp` | 屯門 |
| `yl` | `/icons/hk_18d_yl.webp` | 元朗 |
| `tw` | `/icons/hk_18d_tw.webp` | 荃灣 |
| `kwt` | `/icons/hk_18d_kwt.webp` | 葵青 |
| `island` | `/icons/hk_18d_island.webp` | 離島 |

### Lucide 圖示（kebab-case）

若 `icon` **不以 `/` 開頭**，系統會視為 [Lucide](https://lucide.dev/icons/) 圖示名稱（kebab-case），並自動轉換為元件。

現有地圖範例：

| 組別 | `icon` 值 |
|------|-----------|
| 鐵路 | `tram-front` |
| 公用事業 | `zap` |

其他常用選項：`train-front`、`building-2`、`landmark`、`droplets`、`flame` 等。完整列表請參考 [lucide.dev/icons](https://lucide.dev/icons/)。

---

## 遊戲設定（`config`）

```json
{
  "GO_SALARY": 200,
  "GO_LANDING_BONUS": 100,
  "JAIL_POSITION": 10,
  "JAIL_BAIL": 50,
  "OWNABLE_SPACE_TYPES": ["property", "railroad", "utility"],
  "PLAYER_COLORS": ["#C0DA5A", "#FFC13F", "..."]
}
```

| 欄位 | 說明 |
|------|------|
| `GO_SALARY` | 經過起點獎金 |
| `GO_LANDING_BONUS` | 停留起點額外獎金（經過 + 停留 = 總獎金） |
| `JAIL_POSITION` | 監獄格子 `id`（固定為 10） |
| `JAIL_BAIL` | 保釋金 |
| `OWNABLE_SPACE_TYPES` | 可購買的格子類型 |
| `PLAYER_COLORS` | 玩家棋子顏色（最多 12 色；請與現有地圖使用相同色板） |

> 除非刻意提議規則改動，否則 `config` 數值應與現有地圖保持一致。

---

## 卡牌（`chance` / `community`）

每張卡牌至少需要 `text`（顯示文字）及 `action`（執行動作）。其餘欄位視 `action` 類型而定。

### 卡牌基本格式

```json
{
  "text": "滙豐股息入帳 HK$50。",
  "action": "money",
  "amount": 50
}
```

### 文字佔位符

卡牌 `text` 支援以下佔位符，顯示時會自動替換：

| 佔位符 | 說明 |
|--------|------|
| `{space}` | `move` 卡中替換為 `target` 格子的 `name` |
| `{goSalary}` | 替換為 `config.GO_SALARY`（經過起點獎金） |
| `{goLandingTotal}` | 替換為 `GO_SALARY + GO_LANDING_BONUS`（停留起點總獎金） |

```json
{
  "text": "旺角女人街掃貨，前往{space}。如經過起點，領取 HK${goSalary}。",
  "action": "move",
  "target": 13
}
```

> `target` 必須對應本張地圖 `board` 陣列中有效的格子 `id`（0–39）。

### 動作類型（`action`）

#### `move`：移動至指定格子

| 欄位 | 必填 | 說明 |
|------|------|------|
| `target` | 是 | 目標格子 `id`（0–39） |
| `collectGo` | 否 | 是否計算經過起點獎金。預設 `true`；設為 `false` 則不發放 |

#### `money`：收取或繳付現金

| 欄位 | 必填 | 說明 |
|------|------|------|
| `amount` | 是 | 正數為收入，負數為支出 |

#### `jailCard`：出獄許可證

玩家獲得一張出獄許可證，可於坐監時使用或與其他玩家交易。

#### `jail`：入獄

玩家即時入獄，**不經過起點**，亦**不**領取經過獎金。回合隨即結束。

#### `back`：後退

| 欄位 | 必填 | 說明 |
|------|------|------|
| `spaces` | 是 | 後退格數（正整數） |

#### `nearestRailroad`：前進至最近鐵路

| 欄位 | 必填 | 說明 |
|------|------|------|
| `payDouble` | 否 | 設為 `true` 時，若該鐵路有人擁有，租金加倍 |

#### `repairs`：房屋維修費

| 欄位 | 必填 | 說明 |
|------|------|------|
| `house` | 是 | 每間房屋（1–4 幢）的費用 |
| `hotel` | 是 | 每間酒店（5 幢）的費用 |

#### `payEach` / `collectEach`：向每位玩家付款／收款

| 欄位 | 必填 | 說明 |
|------|------|------|
| `amount` | 是 | 每位玩家的金額 |

#### `typhoon`：打風

觸發打風效果；期間他人支付的租金不付予物主（詳見 [README](../README.md#打風八號十號風球)）。

| 欄位 | 必填 | 說明 |
|------|------|------|
| `signal` | 否 | 風球號數：`8`（持續一輪）或 `10`（持續三輪）。預設 `8` |

#### `rateHike`：加息

觸發加息效果；贖回抵押及建築成本上升 **25%**，持續三輪（詳見 [README](../README.md#加息)）。

### 兩副牌的分工

| 牌組 | 欄位 | 棋盤觸發 |
|------|------|----------|
| 機會卡 | `chance` | 停在 `type: "chance"` 的格子 |
| 社區基金 | `community` | 停在 `type: "community"` 的格子 |

兩副牌**獨立洗牌**，各自維護棄牌堆；抽完後會重新洗牌。現有地圖每副約 **20 張**（含打風、加息卡）；可依主題調整數量。

卡牌金額宜與地圖經濟規模一致（例如「新界各區」的 `GO_SALARY` 較低，卡牌金額亦相應較細）。

---

## 設計檢查清單

- [ ] JSON 放在 `maps/`
- [ ] `board` 陣列長度為 **40**，`id` 從 **0** 到 **39** 連續不重複
- [ ] `id` 0 為 `go`，`id` 10 為 `jail`，`id` 20 為 `parking`，`id` 30 為 `gotojail`
- [ ] 每個 `property` / `railroad` / `utility` 的 `group` 在 `groups` 中有對應定義
- [ ] 每個地產組的 `spaceIds` 與實際格子 `id` 一致
- [ ] 鐵路組固定 4 格（`spaceIds` 長度 4），公用事業固定 2 格
- [ ] 機會（`chance`）及社區基金（`community`）棋盤格各 3 格為佳（與現有地圖一致）
- [ ] 稅項（`tax`）2 格
- [ ] `chance` 及 `community` 陣列已填寫，每張卡都有 `text` 和 `action`
- [ ] `move` 卡的 `target` 在 0–39 範圍內，且指向合理的格子
- [ ] `money` / `payEach` / `collectEach` 設有 `amount`；`back` 設有 `spaces`；`repairs` 設有 `house` 和 `hotel`
- [ ] 卡牌文字中的金額描述與 `amount` / 費率欄位一致
- [ ] 邏輯無意改動行為時，只改 `text` 即可

---

## 提交新地圖

1. 在 `maps/` 新增或編輯 `<your-map-id>.json`（含棋盤與卡牌）
2. 參考 `urban1.json` 或 `rural1.json` 完整結構
3. 開啟 Pull Request 至本 repo，並簡述主題、地區選擇及卡牌本地化構想

我們歡迎其他本地化主題棋盤的創意提案！
