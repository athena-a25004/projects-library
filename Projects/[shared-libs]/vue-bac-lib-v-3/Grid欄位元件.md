---
type: module-note
project: vue-bac-lib-v-3
module: Grid 欄位元件
tags: [vue-bac-lib, grid, components]
---

# Grid 欄位元件

用於 `CoreGridField.gridColumns` 陣列內，定義 Syncfusion Grid 的欄位呈現與互動行為。與 [[UI元件]] 的 `Bac*` Class 不同，這套專門在 Grid 的 cell 中使用。

## 欄位元件一覽

| Class | 說明 |
|-------|------|
| `GridTextBox` | 文字輸入欄位 |
| `GridTextBoxWithBtn` | 文字輸入 + 按鈕（開啟輔助選擇） |
| `GridNumberBox` | 數字輸入 |
| `GridPrice` | 價格顯示（千分位） |
| `GridPriceWithBtn` | 價格 + 按鈕 |
| `GridPercentage` | 百分比欄位 |
| `GridDatePicker` | 日期選擇 |
| `GridDateTimePicker` | 日期時間選擇 |
| `GridTimePicker` | 時間選擇 |
| `GridDropdownList` | 單選下拉 |
| `GridDropdownListWithBtn` | 下拉 + 按鈕 |
| `GridDropdownListWithHyperLink` | 下拉 + 超連結 |
| `GridDropdownYN` | 是/否下拉 |
| `GridDropdownTree` | 樹狀下拉 |
| `GridMultipleDropdown` | 多選下拉 |
| `GridMultipleChipDropdown` | 多選 Chip 下拉（標籤顯示） |
| `GridSelectGrid` | Grid 選擇器 |
| `GridMultipleSelectGrid` | 多選 Grid 選擇器 |
| `GridSelectGridButton` | Grid 選擇 + 按鈕 |
| `GridSelectGridWithBtn` | Grid 選擇 + 自訂按鈕 |
| `GridCheckBox` | 核取方塊 |
| `GridHyperLinkText` | 超連結文字 |
| `GridColorPick` | 顏色選擇 |
| `GridLocaleEditor` | 多語系文字欄位 |
| `GridLocaleObjectEditor` | 多語系物件欄位 |
| `GridCommandButton` | 操作按鈕（刪除、編輯等命令按鈕） |

## 使用方式

```ts
import { CoreGridField, GridTextBox, GridDropdownList, GridCommandButton } from 'vue-bac-lib'

class MyPage extends CoreGridField {
  gridColumns = [
    new GridTextBox({
      field: 'name',
      headerText: '名稱',
      width: 150,
    }),
    new GridDropdownList({
      field: 'status',
      headerText: '狀態',
      width: 100,
      dataSource: [{ value: '1', text: '啟用' }],
    }),
    new GridCommandButton({
      headerText: '操作',
      commands: [
        { type: 'Edit', buttonOption: { iconCss: 'e-icons e-edit' } },
        { type: 'Delete', buttonOption: { iconCss: 'e-icons e-delete' } },
      ],
    }),
  ]
}
```

## 常見 Props（通用）

| Prop | 說明 |
|------|------|
| `field` | 對應資料的 key |
| `headerText` | 欄位標題 |
| `width` | 欄位寬度（px 或百分比） |
| `visible` | 是否顯示 |
| `allowEditing` | 是否可編輯（預設 true） |
| `textAlign` | 文字對齊（`Left`、`Center`、`Right`） |
| `format` | 顯示格式（日期、數字） |

## 注意事項

- `GridCommandButton` 的 `type` 若為 Syncfusion 內建命令（`Edit`/`Delete`/`Save`/`Cancel`），會自動與 Grid 的 editing 功能整合
- 多選系列（`GridMultipleDropdown`、`GridMultipleChipDropdown`）的值通常是陣列，存到 DB 前需序列化

## 相關連結

- [[Core類#CoreGridField — Grid 欄位]]
- [[UI元件]]（表單欄位，不是 Grid 內使用）
