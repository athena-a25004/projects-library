---
type: module-note
project: vue-bac-lib-v-3
module: 套件-API速查
tags: [vue-bac-lib, api, useTools, useDialog, context]
---

# API 速查

← [[套件使用說明]]

---

## useTools

```ts
import { useTools } from 'vue-bac-lib'

// Toast 通知
useTools().bacToast('操作成功', { type: 'success' })   // success / warning / error / info
useTools().bacToast('操作失敗', { type: 'error' })

// Alert 對話框（無回傳值）
useTools().bacAlert('發生錯誤訊息')
useTools().bacAlert(['錯誤 1', '錯誤 2'])    // 多筆錯誤

// Confirm 確認對話框（回傳 Promise<boolean>）
const isConfirm = await useTools().bacConfirm('確定要取消嗎？')
if (!isConfirm) return

// i18n
const t = (key: string) => useTools().i18n.t(key).toString()
t('programs.CRS0110010.title')
```

---

## useDialog

```ts
import { useDialog } from 'vue-bac-lib'

// 從 factory function 建立 Dialog
const dialog = useDialog()
  .createDialogFromCores(
    () => CRS0110011(prgId, id),    // 回傳 cores 的 factory function
    {
      // 對應子程式 cores.main.getContext()?.emit('xxx') 的 callbacks
      save:   () => { dialog.close() },
      create: () => { dialog.close(); openNew() },
      copy:   (formData) => { dialog.close(); openNew(formData) }
    }
  )
  .onBeforeClose(() => {
    cores.searchCore.doSearch()    // 關閉前重查
  })
  .onClose(() => {
    // 完全關閉後執行
  })

dialog.close()    // 手動關閉

// 掛載任意 Vue 元件（不走 factory）
const instance = useDialog().createRootComponent(
  defineComponent({ setup() { return () => h(MyComponent, props) } })
)
instance.destroy()
```

### 子程式觸發父層 callback

```ts
// 子程式（CRS0110011）內
cores.main.getContext()?.emit('create')         // 觸發父層 create callback
cores.main.getContext()?.emit('copy', formData) // 帶參數
```

---

## Grid context 常用 API

透過 `cores.gridCore` 存取（在 factory closure 或 callback 內）：

```ts
// 載入資料
cores.gridCore.getAction().context.loadData(data)              // 前端分頁
cores.gridCore.getAction().context.loadData(data, false, total) // 後端分頁

// 或在 setupPage / created 裡直接用 context（已初始化後才可用）
cores.gridCore.context?.loadData(data)

// 取得所有資料
const all = cores.gridCore.context?.getAllData()

// 取得已勾選列
const selected = cores.gridCore.context?.getSelectedRecords()

// 取得單列
const row = cores.gridCore.context?.getRowDataByIndex(0)

// Excel 匯出
cores.gridCore.getAction().context.excelExport({ fileName: '匯出.xlsx' })

// 新增列（allowEditing: true 時）
cores.gridCore.context?.addRecord()
```

---

## 表單 context 常用 API

透過 `cores.fieldCore.context` 存取：

```ts
// 取得所有欄位值（物件）
const form = cores.fieldCore.context.getFieldsData()

// 填入資料到所有欄位
cores.fieldCore.context.setFieldsData(apiResponse)

// 取得 handler（操作個別欄位）
const handler = cores.fieldCore.context.getHandler()
handler.name.component.setValue('新值')
const val = handler.name.component.getValue()

// 設定下拉選項
handler.status.component.setOptions([{ text: 'A', value: 'a' }])

// 設定欄位可見性 / 禁用
handler.name.setVisible(false)
handler.name.setDisabled(true)

// 驗證
const result = cores.fieldCore.context.getValidationService().validate()
if (!result.success) {
  result.messages.forEach(m => useTools().bacAlert(m))
}

// 用 computed 響應式追蹤驗證狀態
const isValid = computed(() =>
  cores.fieldCore.context?.getValidationService().validate().success
)

// 註冊自訂驗證規則
if (isValidationField<string>(handler.phone))
  cores.fieldCore.context.getValidationService().register(handler.phone, [
    (val?: string) => /^\d{7,15}$/.test(val ?? ''),
    { errorCode: 'Phone Format Error' }
  ])
```

---

## 各 Factory 入口（via modelFactory）

所有 factory 透過 `modelFactory` 物件統一存取：

```ts
import { modelFactory } from 'vue-bac-lib'

// 常用 factory
modelFactory.searchAndGridFactory(actions, prgId)          // 列表頁
modelFactory.searchAndGridDialogFactory(actions, prgId)    // 列表頁（Grid 在 Dialog 內）
modelFactory.singleAndGridFactory(actions, prgId)          // 表單 + Grid（非 Dialog）
modelFactory.singleAndGridDialogFactory(actions, prgId)    // 表單 + Grid（Dialog）
modelFactory.setupFactory(actions, prgId)                  // 設定頁
modelFactory.doubleGridFactory(actions, prgId)             // 雙 Grid
modelFactory.datagridFactory(actions, prgId)               // 單純 Grid
modelFactory.singleFieldFactory(actions, prgId)            // 單純表單

// 轉成 Vue 元件（頁面進入點用）
modelFactory.componentFactory(coreFactory)
```

> 完整 37 個 Model 清單見 [[Model速查]]；各 factory 的 action props 見 [[Model實作指南]]。

---

## searchConditionToQuery

```ts
import { searchConditionToQuery } from 'vue-bac-lib'

// search callback 可接收 conditions 參數
search: {
  context: undefined as never,
  async search(conditions) {
    const qs = searchConditionToQuery(conditions)
    const { data } = await $fetch(`/api/items?${qs}`)
    cores.gridCore.getAction().context.loadData(data)
  }
}
```

---

## isValidationField

```ts
import { isValidationField } from 'vue-bac-lib'

// 型別守衛，確認欄位支援驗證後才呼叫 register
if (isValidationField<string>(handler.phone)) {
  context.getValidationService().register(handler.phone, [
    (val?: string) => /^\d{7,15}$/.test(val ?? ''),
    { errorCode: 'Phone Format Error' }
  ])
}
```
