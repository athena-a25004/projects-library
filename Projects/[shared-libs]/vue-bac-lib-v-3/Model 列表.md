---
type: module-note
project: vue-bac-lib-v-3
module: Model 速查
tags: [vue-bac-lib, model, layout]
---

# Model 速查

37 個預定義 Model，由 `modelFactory` 工廠產生，提供標準化的頁面佈局。搭配 [[Auto引擎#BRenderer]] 使用，Core 類 setup 後注入 Model，BRenderer 依 Model 渲染 UI。

## 取得 Model

```ts
import { modelFactory } from 'vue-bac-lib'

// 在 Core class 的 setup() 內
const model = modelFactory('searchAndGrid')
```

或使用 Layout 工具函數：

```ts
import { getLayout, getGridLayout } from 'vue-bac-lib'
```

---

## Model 分類一覽

### 🔍 搜尋 + 表格 系列

| Model 名稱 | 說明 |
|-----------|------|
| `searchAndGrid` | 標準搜尋 + Grid（最常用） |
| `searchAndGridSingle` | 搜尋 + Grid + 右側單筆表單 |
| `searchAndSingleGrid` | 搜尋 + 單筆 + Grid（上下佈局） |
| `searchAndGridConfirmDialog` | 搜尋 + Grid + 確認對話框 |

### 📊 搜尋 + 樞紐分析 系列

| Model 名稱 | 說明 |
|-----------|------|
| `searchAndPivotView` | 搜尋 + PivotView |
| `singleAndPivotView` | 單筆 + PivotView |
| `toolbarAndPivotView` | Toolbar + PivotView |

### 📋 表格 系列

| Model 名稱 | 說明 |
|-----------|------|
| `datagrid` | 純 Grid（無搜尋） |
| `dialogGrid` | 對話框內的 Grid |
| `doubleGrid` | 雙 Grid（主從關係） |
| `doubleGridSetup` | 雙 Grid + 設定 |
| `gridAndSingle` | Grid + 單筆表單 |
| `singleAndGrid` | 單筆表單 + Grid |
| `singleAndDobuleGrid` | 單筆 + 雙 Grid |

### 📝 表單確認 系列

| Model 名稱 | 說明 |
|-----------|------|
| `datagridPageCommonConfirmDialog` | Grid 頁面通用確認對話框 |
| `datagridPageConfirmDialog` | Grid 頁面確認對話框 |
| `singleGridConfirmDialog` | 單 Grid 確認對話框 |
| `doubleGridConfirmDialog` | 雙 Grid 確認對話框 |
| `searchAndGridConfirmDialog` | 搜尋 + Grid 確認對話框 |
| `singlePageConfirmDialog` | 單頁確認對話框 |
| `singlePageDialog` | 單頁對話框 |
| `tabConfirmDialog` | Tab 確認對話框 |

### 🔧 Toolbar 系列

| Model 名稱 | 說明 |
|-----------|------|
| `toolbar` | 單純 Toolbar |
| `toolbarAndDashboard` | Toolbar + Dashboard |
| `toolbarAndImageUploader` | Toolbar + 圖片上傳 |

### 📑 Tab 系列

| Model 名稱 | 說明 |
|-----------|------|
| `singleAndTab` | 單筆 + Tab |
| `tabAndDialog` | Tab + 對話框 |
| `tabsArea` | 多 Tab 區域 |

### 📌 單筆 / 設定 系列

| Model 名稱 | 說明 |
|-----------|------|
| `singlefield` | 單一表單欄位 |
| `setup` | 設定頁（搭配 CoreSetup） |
| `report` | 報表頁 |

### 🖼️ 特殊 系列

| Model 名稱 | 說明 |
|-----------|------|
| `dashboard` | Dashboard 儀表板 |
| `pivotView` | PivotView |
| `handWrite` | 手寫簽名 |
| `listBoxAndGrid` | ListBox + Grid |
| `multipleImageUploader` | 多圖片上傳 |
| `search` | 純搜尋條件 |

---

## 典型使用範例

### searchAndGrid（最常見）

```ts
class ReservationList extends CoreSearchAndGrid {
  model = modelFactory('searchAndGrid')

  searchFields = [
    new BacDateRangePicker({ label: '日期', field: ['startDate', 'endDate'] }),
    new BacDropdownlist({ label: '狀態', field: 'status', dataSource: statusOptions }),
  ]

  gridColumns = [
    new GridTextBox({ field: 'name', headerText: '名稱' }),
    new GridDatePicker({ field: 'date', headerText: '日期' }),
  ]

  async fetchData(condition: any) {
    return await reservationApi.list(condition)
  }
}
```

### doubleGrid（主從 Grid）

```ts
class OrderPage extends CoreGridField {
  model = modelFactory('doubleGrid')
  // 主 Grid（訂單）+ 從 Grid（明細）
}
```

---

## 相關連結

- [[Core類]]（Model 與 Core 類搭配使用）
- [[Auto引擎]]（BRenderer 依 Model 渲染）
