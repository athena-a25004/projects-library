---
type: module-note
project: vue-bac-lib-v-3
module: UI 元件
tags: [vue-bac-lib, components, ui]
---

# UI 元件

34 個 UI 元件，每個元件有兩個匯出：
- **Vue 元件**：`B` 前綴，直接在 template 使用（如 `<BTextBox>`）
- **TS Class**：`Bac` 前綴，在 Core 類或 Field 陣列中宣告（如 `new BacTextBox({...})`）

## 輸入類

| Vue 元件 | TS Class | 說明 |
|---------|---------|------|
| `BTextBox` | `BacTextBox` | 文字輸入 |
| — | `BacPasswordBox` | 密碼輸入 |
| — | `BacReadyOnlyTextBox` | 唯讀文字 |
| — | `BacTextAreaBox` | 多行文字 |
| `BMoneyInput` | `BacMoneyInput` | 金額輸入（千分位、小數位） |
| `BNumberBox` | `BacNumberBox` | 數字輸入 |
| — | `BacPrice` | 價格顯示 |
| — | `BacReadonlyPrice` | 唯讀價格 |

## 日期 / 時間類

| Vue 元件 | TS Class | 說明 |
|---------|---------|------|
| `BDatePicker` | `BacDatePicker` | 日期選擇 |
| `BShiftDatePicker` | `BacShiftDatePicker` | 班別日期選擇 |
| `BTimePicker` | `BacTimePicker` | 時間選擇 |
| `BDateRangePicker` | `BacDateRangePicker` | 日期範圍選擇 |
| `BDateTimePicker` | `BacDateTimePicker` | 日期時間選擇 |
| `BMonthWithDate` | `BacMonthWithDate` | 月份含日期 |

## 下拉選單類

| Vue 元件 | TS Class | 說明 |
|---------|---------|------|
| `BDropdownlist` | `BacDropdownlist` | 單選下拉 |
| — | `BacDropdownYN` | 是/否下拉（固定選項） |
| `BMultiDropdownlist` | `BacMultipleDropDownList` | 多選下拉 |
| `BMultiDropdownlistAdvance` | `BacMultipleSelectGridAdvance` | 進階多選（含搜尋） |
| `BDropdownTree` | `BacDropdownTree` | 樹狀下拉 |
| — | `BacSelectGrid` | 下拉 Grid 選擇 |
| — | `BacMultipleSelectGrid` | 多選 Grid |

## 按鈕類

| Vue 元件 | TS Class | 說明 |
|---------|---------|------|
| `BButton` | `BacButton` | 一般按鈕 |
| `BUploadButton` | `BacUploadButton` | 上傳按鈕 |
| `BDropdownButton` | `BacDropdownButton` | 下拉選單按鈕 |
| `BSplitButton` | `BacSplitButton` | 分割按鈕 |

## 核取方塊 / 單選類

| Vue 元件 | TS Class | 說明 |
|---------|---------|------|
| `BCheckbox` | `BacCheckbox` | 核取方塊 |
| `BGroupCheckbox` | `BacGroupCheckBox` | 群組核取方塊 |
| `BRadioButton` | `BacRadioButton` | 單選按鈕 |

## EJ2 整合元件

| Vue 元件 | TS Class | 說明 |
|---------|---------|------|
| `ej2-grid`（內部） | — | Syncfusion Grid（透過 Core 使用） |
| `ej2-pivot-table`（內部） | — | Syncfusion PivotView |
| `BSchedule` | `BacSchedule` | Syncfusion Schedule（行程表） |
| `BPdfViewer` | `BacPdfViewer` | PDF 檢視器 |
| `Dashboard` | — | Syncfusion Dashboard Layout |

## 其他元件

| Vue 元件 | TS Class | 說明 |
|---------|---------|------|
| `BColorPick` | `BacColorPick` | 顏色選擇器 |
| `BLocaleEditor` | `BacLocaleEditor` | 多語系文字編輯 |
| — | `BacLocaleAreaEditor` | 多語系區塊編輯 |
| `BMessage` | `BacMessage` | 訊息提示 |
| `BIframe` | `BacIframe` | 內嵌 iframe |
| `BMoreOption` | `BacMoreOption` | 更多選項 |
| `BVhtml` | `BacVhtml` | HTML 渲染 |
| `BWithOperator` | `BacWithOperator` | 帶運算子的欄位（搜尋條件用） |
| `BListbox` | `BacListBox` | 清單選擇 |
| `BImgComponent` | `BacImage` | 圖片元件（手寫簽名） |
| `BCompoundComponent` | `BacCompoundComponent` | 複合元件 |

## 常見使用模式

### 在 Field 陣列中宣告

```ts
// Core 類內部
searchFields = [
  new BacDropdownlist({
    label: '狀態',
    field: 'status',
    dataSource: [{ value: '1', text: '啟用' }]
  }),
  new BacDateRangePicker({
    label: '日期範圍',
    field: ['startDate', 'endDate']
  }),
]
```

### 直接在 Template 使用

```vue
<BTextBox v-model="value" placeholder="請輸入" />
<BDropdownlist v-model="selected" :dataSource="options" />
```

## 相關連結

- [[Grid欄位元件]]（Grid 內嵌的欄位元件，另有一套）
- [[Core類]]（Field 陣列在 Core 類中使用）
