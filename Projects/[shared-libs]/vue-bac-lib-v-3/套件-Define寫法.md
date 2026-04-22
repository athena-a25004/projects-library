---
type: module-note
project: vue-bac-lib-v-3
module: 套件-Define寫法
tags: [vue-bac-lib, fieldDefine, gridDefine, toolbarDefine]
---

# Define 檔案寫法

← [[套件使用說明]]

---

## fieldDefine.ts — 搜尋／表單欄位

```ts
// app/program/CRS0110011/fieldDefine.ts
import {
  BacTextBox, BacTextAreaBox, BacDropdownlist, BacDatePicker,
  getGenLayout
} from 'vue-bac-lib'

// 接受 prg_id 參數方便 i18n key 組合，也方便複用
export default (prg_id: string) =>
  getGenLayout(() => ({
    reservationName: {
      component: new BacTextBox(),
      rowIndex: 0, colIndex: 0, colSize: 4,
      label: `${prg_id}reservationName`,
      required: true
    },
    status: {
      component: new BacDropdownlist<string>({
        fields: { text: 'name', value: 'value' },
        showClearButton: false
      }),
      rowIndex: 0, colIndex: 1, colSize: 2,
      label: `${prg_id}status`,
      disabled: true    // 唯讀欄位
    },
    remarks: {
      component: new BacTextAreaBox(),
      rowIndex: 1, colIndex: 0, colSize: 16,
      rowSize: 2,        // 佔 2 列高度
      label: `${prg_id}remarks`
    }
  }))()    // ← 結尾呼叫 ()，回傳 layout 物件而非函式
```

### 搜尋欄位（genFieldLayout）— 與表單欄位寫法相同

搜尋區塊也用 `getGenLayout`，差別在於 `index.ts` 的 `field.getLayout()` 直接呼叫這個函式：

```ts
// app/program/CRS0110010/fieldDefine.ts
import { BacTextBox, BacDropdownlist, getGenLayout } from 'vue-bac-lib'

export function genFieldLayout() {
  return getGenLayout(() => ({
    keyword: {
      component: new BacTextBox(),
      rowIndex: 0, colIndex: 0, colSize: 4,
      label: 'programs.CRS0110010.keyword'
    },
    status: {
      component: new BacDropdownlist<string>({
        fields: { text: 'text', value: 'value' }
      }),
      rowIndex: 0, colIndex: 1, colSize: 2,
      label: 'programs.CRS0110010.status'
    }
  }))()
}
```

### getGenLayout 常用 Props

| Prop | 型別 | 說明 |
|------|------|------|
| `component` | `Bac*` 實例 | 欄位元件 |
| `rowIndex` | `number` | 列索引 |
| `colIndex` | `number` | 行索引（每列預設 16 格） |
| `colSize` | `number` | 佔幾格（1–16） |
| `rowSize` | `number` | 佔幾列高（預設 1） |
| `label` | `string` | i18n key |
| `required` | `boolean` | 是否必填 |
| `disabled` | `boolean` | 是否禁用 |

---

## gridDefine.ts — Grid 欄位

```ts
// app/program/CRS0110011/gridDefine.ts
import {
  GridTextBox, GridDateTimePicker, GridDropdownList,
  getGridLayout
} from 'vue-bac-lib'

export default (prg: string) =>
  getGridLayout(() => ({
    uuid: {
      component: new GridTextBox(),
      colIndex: 0,
      label: '',
      maxLength: 0,
      isPrimaryKey: true,
      visible: false,    // 不顯示主鍵欄
      required: true,
      disabled: true
    },
    name: {
      component: new GridTextBox(),
      colIndex: 1,
      label: `${prg}name`,
      maxLength: 100,
      colSize: 150,       // 像素寬度
      required: false,
      isPrimaryKey: false
    },
    status: {
      component: new GridDropdownList<string>({
        options: [
          { text: 'NORMAL', value: 'NORMAL' },
          { text: 'CANCEL', value: 'CANCEL' }
        ],
        fields: { text: 'text', value: 'value' }
      }),
      colIndex: 2,
      label: `${prg}status`,
      colSize: 100,
      required: false,
      isPrimaryKey: false
    },
    beginDate: {
      component: new GridDateTimePicker(),
      colIndex: 3,
      label: `${prg}beginDate`,
      colSize: 120,
      required: false,
      isPrimaryKey: false
    }
  }))()    // ← 同上，結尾 ()
```

### getGridLayout 常用 Props

| Prop | 型別 | 說明 |
|------|------|------|
| `component` | `Grid*` 實例 | 欄位元件 |
| `colIndex` | `number` | 欄位順序 |
| `label` | `string` | i18n key |
| `colSize` | `number` | 欄位像素寬度 |
| `maxLength` | `number` | 最大長度（GridTextBox） |
| `isPrimaryKey` | `boolean` | 是否為主鍵 |
| `visible` | `boolean` | 是否顯示（預設 true） |
| `required` | `boolean` | 必填 |
| `disabled` | `boolean` | 禁用 |

---

## toolbarDefine.ts — Toolbar 按鈕

```ts
// app/program/CRS0110011/toolbarDefine.ts
import { BacButton, BacDropdownButton, getGenToolbarLayout } from 'vue-bac-lib'

export function getSingleLeftToolbarLayout() {
  return getGenToolbarLayout(() => ({
    create: {
      component: new BacDropdownButton({
        caption: 'programs.XXX.create',
        items: [
          { text: '選項 A', id: 'A' },
          { text: '選項 B', id: 'B' }
        ]
      }),
      colIndex: 0,
      minWidth: 150
    },
    cancel: {
      component: new BacButton({
        caption: 'programs.XXX.cancel',
        cssClassArray: ['!border-danger']    // Tailwind + !important
      }),
      colIndex: 1,
      visible: false    // 預設隱藏，透過 handler.cancel.setVisible(true) 顯示
    }
  }))()
}

export function getSingleRightToolbarLayout() {
  return getGenToolbarLayout(() => ({
    saveAndCreate: {
      component: new BacButton({ caption: 'programs.XXX.saveAndCreate' }),
      colIndex: 0
    }
  }))()
}
```

### 簡單 Toolbar 可直接內聯

不需要另開 `toolbarDefine.ts`，可直接在 `index.ts` 的 `getLayout()` 內寫：

```ts
toolbarLeft: {
  context: undefined as never,
  getLayout() {
    return getGenToolbarLayout(() => ({
      createBtn: {
        component: new BacButton({ caption: 'programs.CRS.create' }),
        colIndex: 0
      }
    }))()    // ← 結尾呼叫兩次 ()
  }
}
```

### getGenToolbarLayout 常用 Props

| Prop | 型別 | 說明 |
|------|------|------|
| `component` | `BacButton` / `BacDropdownButton` 等 | 按鈕元件 |
| `colIndex` | `number` | 排列順序 |
| `minWidth` | `number` | 最小寬度（px） |
| `visible` | `boolean` | 是否顯示（預設 true） |

---

## 雙括號 `()()` 說明

`getGenLayout`、`getGridLayout`、`getGenToolbarLayout` 都回傳的是**函式**：

```ts
// 回傳函式（不加第二個 ()）— 適合在 factory 的 getLayout/getFields 中使用
getLayout() {
  return getGenLayout(...)    // 回傳 () => layout，factory 內部會呼叫它
}

// 回傳 layout 物件（加第二個 ()）— 在 define 檔案直接回傳時使用
export default () => getGenLayout(...)()   // 立即執行，回傳 layout 物件
```

crs 慣例：**define 檔案直接呼叫兩次 `()()`**，`index.ts` 的 `getLayout/getFields` 直接回傳呼叫結果。
