# 地圖編輯指南

歡迎為 HKOPOLY 設計新棋盤！本指南說明地圖 JSON 的資料結構，以及建立或修改地圖時需要注意的事項。

## 內容位置

地圖 JSON 存放於 `maps/`。遊戲網站：[hkopoly.com](https://hkopoly.com)。

參考檔案：`maps/urban1.json`、`maps/rural1.json`。

---

## 檔案結構

每個地圖 JSON 包含五個頂層欄位：

| 欄位 | 說明 |
|------|------|
| `id` | 地圖唯一識別碼（例如 `urban1`） |
| `name` | 顯示名稱（例如 `市區 1`） |
| `config` | 遊戲規則設定（共用數值；見下方） |
| `board` | 40 格棋盤定義（`id` 0–39） |
| `groups` | 地產顏色組定義 |

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

## 設計檢查清單

- [ ] JSON 放在 `maps/`
- [ ] `board` 陣列長度為 **40**，`id` 從 **0** 到 **39** 連續不重複
- [ ] `id` 0 為 `go`，`id` 10 為 `jail`，`id` 20 為 `parking`，`id` 30 為 `gotojail`
- [ ] 每個 `property` / `railroad` / `utility` 的 `group` 在 `groups` 中有對應定義
- [ ] 每個地產組的 `spaceIds` 與實際格子 `id` 一致
- [ ] 鐵路組固定 4 格（`spaceIds` 長度 4），公用事業固定 2 格
- [ ] 機會（`chance`）及社區基金（`community`）各 3 格為佳（與現有地圖一致）
- [ ] 稅項（`tax`）2 格

---

## 提交新地圖

1. 在 `maps/` 新增 `<your-map-id>.json`
2. 參考該資料夾內的 `urban1.json` 或 `rural1.json` 完整結構
3. 開啟 Pull Request 至本 repo，並簡述主題與地區選擇

我們歡迎其他本地化主題棋盤的創意提案！
