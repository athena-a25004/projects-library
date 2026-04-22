---
type: module-note
project: vue-bac-lib-v-3
module: 套件-Factory範例
tags: [vue-bac-lib, factory, searchAndGrid, singleAndGrid]
---

# Factory 範例

← [[套件使用說明]]

---

## searchAndGridFactory — 列表頁

`index.ts` 的完整結構：

```ts
// app/program/CRS0110010/index.ts
import { modelFactory, useDialog, getGenToolbarLayout, BacButton } from 'vue-bac-lib'
import { genFieldLayout } from './fieldDefine'
import { gridLayout } from './gridDefine'

export const PRG_ID = 'CRS0110010'

export default function () {
  const cores = modelFactory.searchAndGridFactory(
    {
      // ── main ────────────────────────────────────────
      main: {
        context: undefined as never,
        customQuerySetOption() {
          return { enableQuerySets: true, allowReordering: true }
        }
      },

      // ── field：搜尋欄位 ──────────────────────────────
      field: {
        context: undefined as never,
        getLayout() { return genFieldLayout() },
        async initialPage() {
          const handler = this.context.getHandler()
          const options = await fetchOptions()
          handler.status.component.setOptions(options)
        }
      },

      // ── search：搜尋觸發 ─────────────────────────────
      search: {
        context: undefined as never,
        async search() {
          // 取得搜尋欄位所有值
          const param = cores.searchCoreField.context.getFieldsData()
          const data = await api.list(param)
          cores.gridCore.getAction().context.loadData(data)
        }
        // 或接收 conditions 參數搭配 searchConditionToQuery：
        // async search(conditions) {
        //   const qs = searchConditionToQuery(conditions)
        //   const { data } = await $fetch(`/api/items?${qs}`)
        //   cores.gridCore.getAction().context.loadData(data)
        // }
      },

      // ── grid：Grid ───────────────────────────────────
      grid: {
        context: undefined as never,
        getFields() { return gridLayout() },
        customGridOption() {
          return {
            isShowCheckBox: true,
            allowEditing: true,
            showButtons: []   // 空陣列 = 不顯示預設按鈕
          }
        },
        // 點擊列 → 開啟 Dialog（攔截預設 edit 行為）
        async beforeEditEvent(_, row) {
          openDialog(row.id)
          throw new Error('stop process')
        }
      },

      // ── toolbarLeft（可選）───────────────────────────
      toolbarLeft: {
        context: undefined as never,
        created() {
          // created() 適合綁定事件
          this.context.getHandler().createBtn.component.setClickEvent(() => {
            openDialog(null)
          })
        },
        getLayout() {
          // 簡單的 toolbar 可直接內聯，不需要另開 toolbarDefine.ts
          return getGenToolbarLayout(() => ({
            createBtn: {
              component: new BacButton({ caption: 'programs.CRS.create' }),
              colIndex: 0
            }
          }))()   // ← 結尾要呼叫兩次 ()
        }
      }

      // toolbarRight / toolbarTop 同 toolbarLeft
    },
    PRG_ID
  )

  function openDialog(id: string | null) {
    useDialog()
      .createDialogFromCores(() => DetailProgram(PRG_ID, id))
      .onBeforeClose(() => cores.searchCore.doSearch())
  }

  return cores
}
```

### searchAndGridFactory 的 cores 物件

| Key | 說明 |
|-----|------|
| `cores.main` | 主 Core（QuerySets、loading） |
| `cores.searchCore` | 搜尋觸發 Core |
| `cores.searchCoreField` | 搜尋欄位 Core |
| `cores.gridCore` | Grid Core |
| `cores.toolbarTopCore` | Toolbar（頂部，可選） |
| `cores.toolbarRightCore` | Toolbar（右側，可選） |
| `cores.toolbarLeftCore` | Toolbar（左側，可選） |

### `created()` vs `initialPage()`

| | `created()` | `initialPage()` |
|--|-------------|-----------------|
| 執行時機 | 元件建立時立即 | 頁面初始化流程中 |
| 適合用途 | 綁定點擊事件 | 設定初始值、載入非同步資料 |
| crs 慣例 | toolbar 事件綁定 | field 下拉載入 |

---

## singleAndGridDialogFactory — 表單 + Grid 跳窗

```ts
// app/program/CRS0110011/index.ts
import { modelFactory, isValidationField, useDialog, useTools } from 'vue-bac-lib'
import { computed, ref } from 'vue'
import fieldDefine from './fieldDefine'
import gridDefine from './gridDefine'
import { getSingleLeftToolbarLayout, getSingleRightToolbarLayout } from './toolbarDefine'

export const PRG_ID = 'CRS0110011'

export default function (prgId: string, id: string | null) {
  // Vue computed 可直接搭配使用
  const isFormValid = computed(() =>
    cores.fieldCore.context?.getValidationService().validate().success
  )

  const cores = modelFactory.singleAndGridDialogFactory(
    {
      // ── main ────────────────────────────────────────
      main: {
        context: undefined as never,
        getDialogConfig() {
          return {
            title: ref(prgId),
            width: 1400,
            height: 800,
            location: 'center'
          }
        },
        async created() {
          // 隱藏不需要的預設按鈕
          const { save, add } = this.context.getHandler()
          save.setVisible(false)
          add.setVisible(false)
        },
        async setupPage() {
          if (id) {
            const data = await api.getById(id)
            cores.fieldCore.context.setFieldsData(data.form)
            cores.gridCore.context?.loadData(data.details)
          }
        },
        async initialPage() {
          // 有傳 id 就走 setupPage，否則可填入複製資料
          if (!id && copiedData) cores.fieldCore.context.setFieldsData(copiedData)
          else await cores.main.setup()
        }
      },

      // ── field：表單欄位 ──────────────────────────────
      field: {
        context: undefined as never,
        getLayout() { return fieldDefine(`programs.${PRG_ID}.`) },
        async initialPage() {
          const handler = this.context.getHandler()

          // 自訂驗證規則
          if (isValidationField<string>(handler.phone))
            this.context.getValidationService().register(handler.phone, [
              (val?: string) => /^\d{7,15}$/.test(val ?? ''),
              { errorCode: 'Phone Format Error' }
            ])

          // 非同步載入下拉
          const opts = await fetchOptions()
          handler.status.component.setOptions(opts)
          // SelectGrid 需要同時設定 setOptions 與 setEditOptions
          handler.countryCode.component.setOptions(opts)
          handler.countryCode.component.setEditOptions(opts)
        }
      },

      // ── grid：明細 Grid ──────────────────────────────
      grid: {
        context: undefined as never,
        getFields() { return gridDefine(`programs.${PRG_ID}.`) },
        customGridOption() {
          return { isShowCheckBox: true, allowEditing: false }
        }
      },

      // ── toolbarSingleLeft / Right ────────────────────
      toolbarSingleLeft: {
        context: undefined as never,
        created() {
          const { saveBtn } = this.context.getHandler()
          saveBtn.component.computed.disabled.add(() => !isFormValid.value)
          saveBtn.component.setClickEvent(doSave)
        },
        getLayout() { return getSingleLeftToolbarLayout() }
      },
      toolbarSingleRight: {
        context: undefined as never,
        created() {
          const { saveAndCreate } = this.context.getHandler()
          saveAndCreate.component.setClickEvent(async () => {
            await doSave()
            cores.main.getContext()?.emit('create')  // 觸發父層 callback
          })
        },
        getLayout() { return getSingleRightToolbarLayout() }
      }
    },
    prgId
  )

  async function doSave() {
    const form = cores.fieldCore.context.getFieldsData()
    if (id) {
      await api.update(id, form)
    } else {
      const result = await api.create(form)
      id = result.id
      cores.fieldCore.context.setFieldsData(result)
    }
    useTools().bacToast('儲存成功', { type: 'success' })
  }

  return cores
}
```

### singleAndGridDialogFactory 的 cores 物件

| Key | 說明 |
|-----|------|
| `cores.main` | 主 Core（Dialog 狀態） |
| `cores.fieldCore` | 表單欄位 Core |
| `cores.gridCore` | 明細 Grid Core |
| `cores.toolbarSingleLeftCore` | 左側 Toolbar Core |
| `cores.toolbarSingleRightCore` | 右側 Toolbar Core |

### 從父層開啟並接收回傳

```ts
// 父層 index.ts（如 CRS0110010）
function openDetail(id: string | null) {
  const dialog = useDialog()
    .createDialogFromCores(
      () => CRS0110011(PRG_ID, id),
      {
        // 對應子層 cores.main.getContext()?.emit('xxx') 的 callbacks
        create: () => {
          dialog.close()
          openDetail(null)
        },
        copy: (formData) => {
          dialog.close()
          openDetail(null, formData)
        }
      }
    )
    .onBeforeClose(() => cores.searchCore.doSearch())
}
```
