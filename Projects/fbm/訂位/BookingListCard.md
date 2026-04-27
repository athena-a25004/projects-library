---
type: component-spec
project: fbm
module: 訂位
component: BookingListCard
source: app/program/FBM0100000/BookingListCard.vue
version: 7.7.132
tags:
  - fbm
  - reservation
  - card
  - product-spec
---

## 概述

側邊欄訂位摘要卡。出現在所有視圖模式（列表、時間軸、桌圖）的側邊欄，是使用者接觸訂位資料的第一層。點擊後開啟 [[ReviewReserveCard]] 詳情。

## 卡片佈局

```
┌───────────────────────────────────────────────┬──────────┐
│ Row 1: 姓名 + 標籤圖示               時間區  │ 狀態圖示 │
│ Row 2: 電話                       功能圖示列  │  (slot)  │
│ Row 3: 人數（大人/小孩/嬰兒/合計）            │          │
│ Row 4: 桌號標籤 + 鎖桌圖示                    │          │
│ Row 5: 會員卡號（條件顯示）                   │          │
└───────────────────────────────────────────────┴──────────┘
```

左側為資訊區（flex-1），右側為狀態圖示區（由父元件透過 slot 注入 [[狀態系統|StatusIconRender]]）。

---

## Row 1：姓名區 + 時間區

### 姓名區（左）

由左到右依序排列，超過 170px 寬度時 truncate：

| # | 元素 | 顯示規則 | 視覺 |
|---|------|----------|------|
| 1 | 客戶姓名 | `fullName` 有值則顯示；無值顯示「匿名」 | 粗體，max-width 100px truncate |
| 2 | 性別圖示 | M → 藍色男性 / F → 粉色(series-10)女性 / U → 紫色(series-5)跨性別 / S 或 null → 不顯示 | 16px icon |
| 3 | 黑名單圖示 | `customer.isBlocklist = true` | 紅色(danger)禁止圖示 |
| 4 | CRM 會員圖示 | `showMemberCategory = true` 且 `crmSourceSystem` 為 `A7CRM` 或 `A6WS` | 藍色(primary)鑽石圖示 |
| 5 | 外交禮遇圖示 | `customer.isDiplomat = true` | series-7 色政府圖示 |
| 6 | 類型徽章 | 僅 WLB / WLK / WIO 顯示，RSV 不顯示 | 白字 + secondary 底色 pill |

**類型徽章文字：**

| reservationType | 文字 |
|-----------------|------|
| WLB | 候補 |
| WLK | 現場客 |
| WIO | 預約中心 |
| RSV | 不顯示 |

### 時間區（右）

| # | 元素 | 顯示規則 | 視覺 |
|---|------|----------|------|
| 1 | 遲到徽章 | 見 [[#遲到判定]] | 紅底(error)白字「遲到」 |
| 2 | 訂位時間 | `startTime` 有值時顯示 `HH:mm` | 粗體文字 |

---

## Row 2：電話 + 功能圖示

### 電話（左）

- 有手機 → `callingCode` + `mobilePhone`（如 `+886912345678`）
- 無手機 → 「未填手機」

### 功能圖示列（右）

由左到右排列，每個為 20x20 圓形底色 icon，僅在條件成立時出現：

| # | 用途 | 顯示條件 | 圖示/顏色 |
|---|------|----------|-----------|
| 1 | 訂金 | `isDepositRequired = 'Y'` | 已付(PDP) → 綠底綠字錢幣 / 未付 → 黃底黃字錢幣 |
| 2 | 專案 | `projectName` 有值 | 紫底(#7107DC14)紫字(#7107DC)標籤 |
| 3 | PMS 來源 | `reservationSource = 'PMS'` | primary-container 底色飯店圖示 |
| 4 | 備註 | `notes` 有值 | 灰底(on-surface-opacity8)灰字(outline)文件圖示 |
| 5 | 壽星 | `birthday` 有值 且 `showMemberBirthday = true` 且 生日月份 = 當月 | 灰底灰字蛋糕圖示 |

---

## Row 3：人數

固定顯示，無條件渲染。

```
[人物icon] {大人}   [幼兒icon] {小孩}   [嬰兒icon] {嬰兒}   ({合計})
```

合計 = `numberOfAdults` + `numberOfChildren` + `numberOfInfants`。全部使用 primary 色。

---

## Row 4：桌號

| 情境 | 顯示 |
|------|------|
| 有指派桌位 | 每張桌子一個 pill 標籤，顯示 `deskNos`（灰底、實線圓角邊框） |
| 無指派桌位 | 黃色虛線框 placeholder「桌號」 |
| 鎖桌 | `isTableLocked = 'Y'` 時，桌號列尾顯示灰底(gray-500)白色鎖頭圖示 |

---

## Row 5：會員卡號（條件顯示）

- 顯示條件：`showMemberCardNumber = true` 且 `customer.cardNumber` 有值
- 格式：`CRM編號: {cardNumber}`

---

## 右側 Slot：狀態圖示

由父元件注入 StatusIconRender，判定邏輯詳見 [[狀態系統]]。

簡要對照：

| 優先序 | 條件 | 圖示 | 可點擊 |
|--------|------|------|--------|
| 1 | 候補可轉正 (`status = WTC`) | 綠色箭頭 | 是（觸發轉正） |
| 2 | 候位已取消 (`type=WIO + status=CXL`) | 灰色取消 | 否 |
| 3 | No Show (`status=CXL + reason=NSW`) | 灰色 No Show | 否 |
| 4 | 已取消未過期 (`status=CXL + 未過期`) | 紅色復位 | 是（觸發復位） |
| 5 | 已取消已過期 | 灰色取消 | 否 |
| 6 | 已入座 (`onsiteStatus=STD`) | 橘色椅子 + HH:mm | 否 |
| 7 | 已離席 (`onsiteStatus=LFT`) | 灰色門 | 否 |
| 8 | 通知失敗 (`reminder=NDF`) | 灰色錯誤 | 否 |
| 9 | 客人保留 (`reminder=HLD`) | 綠色勾人 | 否 |
| 10 | 已通知 (`reminder=NTF`) | 綠色勾 + HH:mm | 否 |
| 11 | 未通知 (`reminder=NNF`) | 綠色待辦 | 否 |
| 12 | 都不符合 | 空白 | — |

---

## 互動行為

| 行為 | 觸發 | 結果 |
|------|------|------|
| 點擊卡片 | click 整張卡片 | emit `openSingleData(uuid)`，開啟 ReviewReserveCard |
| Hover | 滑鼠移入 | 背景變 `bg-on-surface-opacity8` |
| 選中狀態 | `uuid = selectedUuid` | 持續高亮背景 |
| 拖曳 | drag 卡片 | 可拖至時間軸或桌圖（由父元件 SidebarList 處理） |

---

## 遲到判定

顯示紅色「遲到」徽章的條件，**全部必須同時成立**：

1. `reservationType != WLB`（候補不算遲到）
2. `startTime` 有值
3. `reservationDetailList.length > 0`（有指派桌位）
4. 所有桌位 `onsiteStatus` 都是 `NST`（都還沒入座）
5. 現在時間 - `startTime` > `holdTime` 分鐘

> 候位(WIO)和現場客(WLK)也會判定遲到，只有候補(WLB)例外。

---

## Figma 比對檢查清單

以下是建議用來比對 Figma 的檢查項目：

- [ ] Row 1 姓名區：truncate 行為、icon 順序是否一致
- [ ] Row 1 類型徽章：WIO 在 Figma 上的文字是否也是「預約中心」
- [ ] Row 1 時間區：遲到徽章的位置與時間的相對關係
- [ ] Row 2 功能圖示：順序（訂金→專案→PMS→備註→壽星）是否與 Figma 一致
- [ ] Row 2 訂金圖示：Figma 是否區分已付/未付的顏色
- [ ] Row 3 人數：Figma 是否有相同的三類（大人/小孩/嬰兒）
- [ ] Row 4 桌號：無桌位時的 placeholder 樣式是否一致
- [ ] Row 4 鎖桌圖示：Figma 是否有標記
- [ ] Row 5 會員卡號：Figma 是否有這一行
- [ ] 右側狀態圖示：Figma 有標記幾種狀態？是否涵蓋全部 12 種
- [ ] 遲到判定：Figma 對遲到的定義是否與程式碼的五個條件一致
- [ ] 選中/Hover 狀態：Figma 是否有標記

---

## 相關連結

- [[訂位]] — 模組總覽
- [[狀態系統]] — StatusIconRender 完整優先序
- [[資料模型]] — ReservationData 完整欄位定義
- 原始碼：`app/program/FBM0100000/BookingListCard.vue`
