---
type: module-note
project: vue-bac-lib-v-3
module: Model 實作指南
tags: [vue-bac-lib, model, factory, how-to]
---

# Model 實作指南

如何用 Factory 系統實作一個標準功能頁面。

---

## 整體流程

```
1. action.ts         呼叫 factory，定義業務邏輯（搜尋、CRUD、事件）
2. genFieldDefine.ts 定義搜尋欄位（getGenLayout）
3. genGridFieldDefine.ts 定義 Grid 欄位（getGridLayout）
4. genToolbarDefine.ts   定義 Toolbar 按鈕（getGenToolbarLayout）
        ↓
componentFactory(actionFn) → Vue 元件
        ↓
<YourPage :cores="cores" />
```

每個 factory 最終回傳一個包含多個 Core 實例的 `cores` 物件，key 名稱為固定命名（`main`、`gridCore`、`searchCore`…）。

---

## 標準目錄結構

```
YourFeature/
├── action.ts              ← factory 呼叫 + 業務邏輯
├── genFieldDefine.ts      ← 搜尋欄位定義
├── genGridFieldDefine.ts  ← Grid 欄位定義
└── genToolbarDefine.ts    ← Toolbar 按鈕定義
```

---

## Step 1：action.ts

以 `searchAndGridFactory`（最常用）為例：

```ts
// YourFeature/action.ts
import { searchAndGridFactory } from 'vue-bac-lib/model/searchAndGrid/common'
// ⚠️ 不要從 'vue-bac-lib/factorys' 引入，已 deprecated

import { fieldLayout } from './genFieldDefine'
import { genGridFieldLayout } from './genGridFieldDefine'
import { genToolbarLayout } from './genToolbarDefine'

export default function () {
  const core = searchAndGridFactory(
    {
      // ── main：主 Core，QuerySets 等全域設定 ──────────
      main: {
        context: undefined as never,
        customQuerySetOption() {
          return { enableQuerySets: true }  // 啟用查詢集
        }
      },

      // ── field：搜尋欄位 Core ─────────────────────────
      field: {
        context: undefined as never,
        getLayout() {
          return fieldLayout()  // 回傳 genFieldDefine 的 layout
        },
        async initialPage() {
          // 頁面建立後設定初始值
          this.context.getHandler().startDate.component.setValue(new Date())
        }
      },

      // ── search：搜尋觸發 Core ────────────────────────
      search: {
        context: undefined as never,
        async search(condition) {
          // condition 是搜尋欄位收集後的條件物件
          const data = await api.list(condition)
          core.gridCore.getAction().context.loadData(data.records, false, data.total)
        }
      },

      // ── grid：Grid Core ──────────────────────────────
      grid: {
        context: undefined as never,
        getFields() {
          return genGridFieldLayout()
        },
        setupPage() {
          // Grid 掛載後自動觸發搜尋
          core.searchCore.doSearch()
        },
        customGridOption() {
          return {
            allowPaging: true,
            sspMode: false,       // false = 前端分頁, true = 後端分頁
            allowEditing: true,
            isShowCheckBox: false,
            allowReordering: true,
            pageSettings: { pageSize: 20 }
          }
        },
        // 依 row 資料動態控制按鈕狀態
        onRowDataBoundEvent(row) {
          return {
            Delete: { disable: row.status === 'LOCKED' },
            Edit:   { disable: row.status === 'LOCKED' }
          }
        }
      },

      // ── toolbarRight（可選）：右側 Toolbar ───────────
      toolbarRight: {
        context: undefined as never,
        getLayout() {
          return genToolbarLayout()
        },
        initialPage() {
          this.context.getHandler().exportBtn.component.setClickEvent(() => {
            core.gridCore.getAction().context.excelExport({ fileName: '匯出.xlsx' })
          })
        }
      },

      // toolbarLeft / toolbarTop 同 toolbarRight，按需加入
    },
    'PMS0210130'  // programId，用於權限管理
  )

  return core
}
```

### factory 回傳的 core 物件

| Key | 說明 |
|-----|------|
| `core.main` | 主 Core（QuerySets、loading 狀態） |
| `core.searchCore` | 搜尋觸發 Core |
| `core.searchCoreField` | 搜尋欄位 Core |
| `core.gridCore` | Grid Core |
| `core.toolbarTopCore` | Toolbar（頂部，可選） |
| `core.toolbarRightCore` | Toolbar（右側，可選） |
| `core.toolbarLeftCore` | Toolbar（左側，可選） |

---

## Step 2：genFieldDefine.ts — 搜尋欄位

```ts
// YourFeature/genFieldDefine.ts
import { getGenLayout } from 'vue-bac-lib'
import { BacTextBox, BacDropdownlist, BacDateRangePicker, BacWithOperator } from 'vue-bac-lib'

export const fieldLayout = getGenLayout(() => ({

  // 基本文字欄位
  name: {
    component: new BacTextBox(),
    rowIndex: 0,
    colIndex: 0,
    colSize: 3,            // Bootstrap grid（1–12），預設 3
    label: 'programs.XXX.name',
    maxLength: 50,
    required: false
  },

  // 下拉選單
  status: {
    component: new BacDropdownlist({
      fields: { text: 'display', value: 'value' },
      valueDisplayFormat: 'textValue',
      options: [
        { display: 'SystemCommon.All', value: '' },
        { display: 'programs.XXX.status.A', value: 'A' },
        { display: 'programs.XXX.status.B', value: 'B' }
      ],
      translateOptionLocale: true,  // 選項 label 走 i18n
      showClearButton: true,
      value: ''
    }),
    rowIndex: 0,
    colIndex: 1,
    colSize: 3,
    label: 'programs.XXX.status'
  },

  // 日期範圍（BacWithOperator，支援 between / eq 等運算子）
  dateRange: {
    component: new BacWithOperator('date', { operator: 'between' }),
    rowIndex: 0,
    colIndex: 2,
    colSize: 6,
    label: 'programs.XXX.dateRange'
  }

}))
```

### getGenLayout 欄位通用 Props

| Prop | 型別 | 說明 |
|------|------|------|
| `component` | `Bac*` class 實例 | 欄位元件 |
| `rowIndex` | `number` | 所在列（從 0 起） |
| `colIndex` | `number` | 同列內排序（從 0 起） |
| `colSize` | `number` 1–12 | Bootstrap 欄寬，預設 3 |
| `label` | `string` | i18n key 或直接文字 |
| `maxLength` | `number` | 最大輸入長度（驗證用） |
| `required` | `boolean` | 必填（底層自動註冊驗證規則） |
| `disabled` | `boolean` | 靜態唯讀 |
| `visible` | `boolean` | 是否顯示 |

---

## Step 3：genGridFieldDefine.ts — Grid 欄位

```ts
// YourFeature/genGridFieldDefine.ts
import { getGridLayout } from 'vue-bac-lib'
import { GridTextBox, GridDropdownList, GridDatePicker, GridNumberBox, GridPrice } from 'vue-bac-lib'

export const genGridFieldLayout = getGridLayout(() => ({

  // 隱藏的 PK 欄位（必填，底層 Grid 操作需要）
  uuid: {
    component: new GridTextBox(),
    colIndex: 0,
    label: '',
    maxLength: 4000,
    isPrimaryKey: true,
    visible: false,
    required: true,
    disabled: true
  },

  // 一般文字欄
  name: {
    component: new GridTextBox(),
    colIndex: 1,
    label: 'programs.XXX.name',
    maxLength: 100,
    colSize: 150,          // 像素寬度，或 'auto'
    required: true,
    isPrimaryKey: false
  },

  // 下拉欄
  status: {
    component: new GridDropdownList({
      fields: { text: 'display', value: 'value' },
      options: [
        { display: 'programs.XXX.status.A', value: 'A' },
        { display: 'programs.XXX.status.B', value: 'B' }
      ],
      translateOptionLocale: true,
      valueDisplayFormat: 'textOnly',
      showClearButton: false
    }),
    colIndex: 2,
    label: 'programs.XXX.status',
    colSize: 120,
    required: true,
    isPrimaryKey: false
  },

  // 日期欄（唯讀）
  createdAt: {
    component: new GridDatePicker(),
    colIndex: 3,
    label: 'commonFieldName.createdAt',
    colSize: 120,
    required: false,
    disabled: true,
    isPrimaryKey: false
  },

  // 數值欄（帶排序設定）
  amount: {
    component: new GridPrice(),
    colIndex: 4,
    label: 'programs.XXX.amount',
    colSize: 100,
    required: true,
    isPrimaryKey: false,
    sortSettings: {
      initialSort: true,       // 頁面載入時預設排序此欄
      multiSortingIndex: 1     // 多欄排序優先順序（-1 表示不參與）
    }
  }

}))
```

### getGridLayout 欄位通用 Props

| Prop | 型別 | 說明 |
|------|------|------|
| `component` | `Grid*` class 實例 | Grid 內嵌欄位元件 |
| `colIndex` | `number` | 欄位排列順序（從 0 起） |
| `colSize` | `number \| 'auto'` | 像素寬度 |
| `label` | `string` | 欄位標題（i18n key 或直接文字） |
| `maxLength` | `number` | 最大輸入長度（0 = 不限制） |
| `isPrimaryKey` | `boolean` | 是否為主鍵（必填，PK 欄通常 hidden） |
| `required` | `boolean` | 必填 |
| `disabled` | `boolean` | 欄位唯讀 |
| `visible` | `boolean` | 是否顯示（預設 true） |
| `sortSettings.initialSort` | `boolean` | 頁面載入時預設排序 |
| `sortSettings.multiSortingIndex` | `number` | 多欄排序優先順序，-1 不參與 |
| `sortSettings.allowSorting` | `boolean` | 允許點擊排序（預設 true） |

---

## Step 4：genToolbarDefine.ts — Toolbar

```ts
// YourFeature/genToolbarDefine.ts
import { getGenToolbarLayout } from 'vue-bac-lib'
import { BacButton } from 'vue-bac-lib'

export const genToolbarLayout = getGenToolbarLayout(() => ({
  exportBtn: {
    component: new BacButton({ caption: 'SystemCommon.excelExport' }),
    colIndex: 0,
    label: ''
  },
  addBtn: {
    component: new BacButton({
      caption: 'SystemCommon.add',
      cssClassArray: ['e-primary']
    }),
    colIndex: 1,
    label: ''
  }
}))
```

> Toolbar 的點擊事件在 `action.ts` 的 `initialPage()` 裡綁定，不在這裡定義。

---

## Step 5：componentFactory — 轉成 Vue 元件

```ts
// 在父層 Vue 元件的 <script setup> 或 defineComponent 內

import { componentFactory } from 'vue-bac-lib'
import yourFeatureAction from './YourFeature/action'

// 將 action function 包裝成 Vue 元件
const YourFeaturePage = componentFactory(yourFeatureAction)
```

或在 Nuxt 頁面直接使用：

```vue
<script setup lang="ts">
import { componentFactory } from 'vue-bac-lib'
import yourFeatureAction from './action'

const YourFeaturePage = componentFactory(yourFeatureAction)
</script>

<template>
  <YourFeaturePage />
</template>
```

### Dialog 版（跳窗）

如果 factory 使用 `searchAndGridDialogFactory`，`componentFactory` 會自動偵測到 `CoreDialog` 並處理 dirty check。

```ts
import { searchAndGridDialogFactory } from 'vue-bac-lib/model/searchAndGrid/common'

export default function () {
  const core = searchAndGridDialogFactory({ ... }, 'PMS0210130')
  return core
}
```

---

## context.getHandler() — 操作元件實例

在 `initialPage()` / `setupPage()` 等 callback 內，透過 `this.context.getHandler()` 拿到所有欄位的 handler：

```ts
initialPage() {
  const h = this.context.getHandler()

  // 設定值
  h.status.component.setValue('A')

  // 取值
  const val = h.name.component.getValue()

  // 綁定點擊事件
  h.submitBtn.component.setClickEvent(() => {
    // do something
  })

  // 動態控制 disabled（computed，響應式）
  h.name.computed.disabled.add(() => {
    return h.status.component.getValue() === 'LOCKED'
  })

  // 動態控制 visible
  h.advancedField.computed.visible.add(() => {
    return h.showAdvanced.component.getValue() === true
  })
}
```

### setupPage() — 載入初始資料

```ts
async setupPage() {
  // 呼叫 API 取得資料後填入欄位
  const data = await api.getDetail(id)
  this.context.setFieldsData(data)
}
```

---

## 跨 core 操作

factory 回傳的 `core` 物件在 action.ts 的 closure 內可以直接取用：

```ts
export default function () {
  const core = searchAndGridFactory({
    search: {
      context: undefined as never,
      async search(condition) {
        const res = await api.list(condition)
        // 操作 gridCore
        core.gridCore.getAction().context.loadData(res.data, false, res.total)
      }
    },
    toolbarRight: {
      context: undefined as never,
      getLayout() { return genToolbarLayout() },
      initialPage() {
        // 操作 gridCore 的匯出
        this.context.getHandler().exportBtn.component.setClickEvent(() => {
          core.gridCore.getAction().context.excelExport({
            fileName: '資料匯出.xlsx',
            header: {
              headerRows: 1,
              rows: [{ cells: [{ colSpan: 5, value: '資料清單', style: { bold: true } }] }]
            }
          })
        })

        // 手動觸發搜尋
        this.context.getHandler().refreshBtn.component.setClickEvent(() => {
          core.searchCore.doSearch()
        })
      }
    },
    // ...
  }, 'PMS0210130')

  return core
}
```

### 常用跨 core API

| 操作 | 程式碼 |
|------|--------|
| 載入 Grid 資料 | `core.gridCore.getAction().context.loadData(data, isPrepend, total)` |
| 匯出 Excel | `core.gridCore.getAction().context.excelExport({ fileName })` |
| 觸發搜尋 | `core.searchCore.doSearch()` |
| 取得目前搜尋條件 | `core.gridCore.getAction().context.getSspQuery()` |
| 取得報表名稱 | `core.main.getAction().context.getReportName()` |

---

## 各 Model 的 factory import

直接從各 model 的 `common.ts` 引入，不要用 `factorys.ts`（deprecated）：

| Factory 名稱 | Import 路徑 |
|-------------|-------------|
| `searchAndGridFactory` | `vue-bac-lib/model/searchAndGrid/common` |
| `searchAndGridDialogFactory` | `vue-bac-lib/model/searchAndGrid/common` |
| `searchAndGridSingleFactory` | `vue-bac-lib/model/searchAndGridSingle/common` |
| `searchAndSingleGridFactory` | `vue-bac-lib/model/searchAndGridSingle/common` |
| `doubleGridFactory` | `vue-bac-lib/model/doubleGrid/common` |
| `doubleGridDialogFactory` | `vue-bac-lib/model/doubleGrid/common` |
| `singleAndGridFactory` | `vue-bac-lib/model/singleAndGrid/common` |
| `singleAndGridDialogFactory` | `vue-bac-lib/model/singleAndGrid/common` |
| `gridAndSingleFactory` | `vue-bac-lib/model/gridAndSingle/common` |
| `singleAndDobuleGridFactory` | `vue-bac-lib/model/singleAndDobuleGrid/common` |
| `searchAndPivotViewFactory` | `vue-bac-lib/model/searchAndPivotView/common` |
| `singleAndPivotView` | `vue-bac-lib/model/singleAndPivotView/common` |
| `listboxAndGridFactory` | `vue-bac-lib/model/listBoxAndGrid/common` |
| `toolbarAndDashboardFactory` | `vue-bac-lib/model/toolbarAndDashboard/common` |
| `singleFieldFactory` | `vue-bac-lib/model/singlefield/common` |
| `setupFactory` | `vue-bac-lib/model/setup/common` |
| `setupDoubleGridFactory` | `vue-bac-lib/model/doubleGridSetup/common` |
| `reportFactory` | `vue-bac-lib/model/report/common` |
| `handWriteFactory` | `vue-bac-lib/model/handWrite/common` |
| `tabFactory` | `vue-bac-lib/model/tabsArea/common` |
| `tabAndDialog` | `vue-bac-lib/model/tabAndDialog/common` |
| `singleAndTabFactory` | `vue-bac-lib/model/singleAndTab/common` |
| `datagridFactory` | `vue-bac-lib/model/datagrid/common` |
| `dialogGridFactory` | `vue-bac-lib/model/dialogGrid/common` |
| `toolbarFactory` | `vue-bac-lib/model/toolbar/common` |
| `multipleImageUploaderFactory` | `vue-bac-lib/model/multipleImageUploader/common` |

> 各 factory 也有對應的 `Dialog` 版本（如 `searchAndGridDialogFactory`），用於跳窗場景。

---

## 相關連結

- [[Model速查]]（各 Model 佈局預覽）
- [[Grid欄位元件]]（Grid* 元件清單）
- [[UI元件]]（Bac* 元件清單）
- [[Core類]]（factory 內部使用的 Core 類架構）
