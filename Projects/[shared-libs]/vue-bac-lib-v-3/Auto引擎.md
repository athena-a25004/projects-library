---
type: module-note
project: vue-bac-lib-v-3
module: Auto 引擎
tags: [vue-bac-lib, auto, renderer, server-driven-ui, low-code]
---

# Auto 引擎

`src/auto/` — **Server-Driven UI 引擎**，讓後端 API 回傳 JSON Schema，前端直接渲染出完整的搜尋+表格或搜尋+樞紐分析頁面，完全不需要前端寫 Core 類程式碼。

主要使用場景：**bi 報表系統**（報表定義存在 DB，可動態配置，不需前端部署）。

---

## 核心概念：流程總覽

```
後端 DB（報表設定）
        ↓ API
DynamicRenderApiResponse（JSON）
        ↓
   BRenderer（接收 schema prop）
        ↓
   renderer() 判斷 model 類型
        ↓
   動態載入對應 Engine（lazy import）
        ↓
   engine.validate()    ← AJV 驗證 JSON 格式
        ↓
   engine.getCores()    ← 用 JSON 建出 Core 類實例
        ↓             ← LayoutAdapter 將 JSON 轉為 Bac* 元件實例
   asyncComponentFactory → Vue 元件
        ↓
   渲染到畫面
```

---

## BRenderer — 入口元件

`src/auto/render/Renderer.vue`

**正確用法**：接收的是從後端 API 拿到的 `schema`，**不是** Core 類實例：

```vue
<script setup lang="ts">
import { BRenderer } from 'vue-bac-lib'
import type { SchemaDefined } from 'vue-bac-lib'

// schema 從後端 API 取得
const schema = await fetchReportSchema(programId)
</script>

<template>
  <BRenderer :schema="schema" />
</template>
```

`SchemaDefined` 結構：

```ts
type SchemaDefined = {
  id: string
  schemaType: 'MODEL' | 'CONFIG'  // 目前只有 MODEL 被實作
  programId: string | null
  jsonSchema: DynamicRenderApiResponse  // 核心 payload
  // ...其他 metadata
}
```

---

## DynamicRenderApiResponse — JSON Schema 結構

這是後端回傳、驅動整個 UI 的核心 JSON：

```ts
interface DynamicRenderApiResponse {
  modelLayout: {
    model: 'searchAndGrid' | 'searchAndPivot'  // 決定用哪個 Engine
    search: {
      searchResource?: string  // 搜尋 API 路徑
    }
    fieldLayout?: ApiFieldLayout[]   // 搜尋條件欄位清單
    gridLayout?: ApiGridLayout[]     // Grid 欄位清單（searchAndGrid 用）
    pivotLayout?: ApiPivotLayout     // Pivot 設定（searchAndPivot 用）
    toolbar?: ApiFieldLayout[]       // Toolbar 按鈕
  }
  fieldLang: Array<{                 // 多語系翻譯
    field: string
    label: Record<string, string>    // { 'zh-TW': '名稱', 'en-US': 'Name' }
  }>
  selectDatasource: {                // 下拉選單資料來源
    [optionResource: string]: Array<{ key: string; value: string }>
  }
}
```

---

## Engine 系統

目前支援 **2 個 Model**（`src/auto/engines/index.ts`）：

| Model | Engine 類 |
|-------|-----------|
| `searchAndGrid` | `SearchAndGridEngine` |
| `searchAndPivot` | `SearchAndPivotEngine` |

兩者皆繼承 `BaseEngine`，提供：

| 方法 | 說明 |
|------|------|
| `validate()` | 用 AJV 驗證 `modelLayout` 格式是否符合對應 Schema |
| `adapter()` | 呼叫 LayoutAdapter，將 JSON 轉換為 Layout 工廠函數 |
| `getCores()` | 組裝 Core 類實例（搜尋邏輯、Grid 欄位、Toolbar） |
| `createModel()` | validate → getCores，若驗證失敗直接 throw |

### SearchAndGridEngine 做了什麼

`src/auto/engines/searchAndGrid.ts`

1. 透過 `LayoutAdapter` 把 `fieldLayout`（JSON）轉成實際 `BacTextBox`、`BacDropdownlist` 等 class 實例
2. 組裝 `searchAndGridFactory`，建出包含以下 Core 的物件：
   - `main`：主 Core，帶 QuerySets 支援
   - `field`：搜尋欄位 Core
   - `search`：搜尋觸發 Core，呼叫 `search.searchResource` API
   - `grid`：Grid Core，帶 allowResizing / allowPaging / allowEditing=false
   - `toolbarRight`：右側 Toolbar，固定有 ExcelExport 按鈕

---

## LayoutAdapter — JSON → Bac* 元件實例

`src/auto/utils/layoutAdapter.ts`

負責把 JSON 裡的 `{ component: 'BacTextBox', props: { textBoxValue: null } }` 轉成 `new BacTextBox({ value: null })` 等實際物件。

四種轉換方法：
- `fieldLayoutAdapter()` — 搜尋欄位
- `gridLayoutAdapter()` — Grid 欄位
- `pivotLayoutAdapter()` — Pivot 欄位
- `toolbarLayoutAdapter()` — Toolbar 按鈕

翻譯邏輯：欄位的 `label` 優先從 `fieldLang` 取當前語系文字，找不到才用 JSON 內的預設值。

---

## componentFactory — 元件 Props 標準化

`src/auto/utils/componentFactory.ts`

Auto 引擎支援的元件類型（`ComponentType`）：

```
搜尋欄位：BacTextBox, BacNumberBox, BacDropdownlist, BacSelectGrid,
          BacCheckbox, BacDatePicker, BacWithOperatorDate, BacWithOperatorNumber
Grid 欄位：GridTextBox, GridNumberBox, GridPrice, GridCheckbox,
          GridDatePicker, GridDateTimePicker
```

> 注意：Auto 引擎支援的元件是 **整個 UI 元件庫的子集**，沒有的元件類型需要手動寫 Core 類頁面。

---

## AJV Schema 驗證

`src/auto/schemas/`

每個 Model 有獨立的 JSON Schema 檔案：
- `searchAndGrid/schema.ts` — 定義 fieldLayout / gridLayout 格式規則
- `searchAndPivot/schema.ts` — 定義 fieldLayout / pivotLayout 格式規則

驗證失敗的錯誤訊息格式：`instancePath: message`（如 `/fieldLayout/0/field: must be string`）

---

## 注意事項

- Auto 引擎的 Model（`searchAndGrid` / `searchAndPivot`）與 [[Model 列表]] 裡的 37 個 Model **不是同一套系統**：37 個 Model 是給手寫 Core 類頁面用的佈局工廠，Auto 引擎是給 Server-Driven UI 用的
- Grid 在 Auto 引擎中固定 `allowEditing: false`、`sspMode: false`（前端分頁），適合唯讀報表頁面
- 若需要可編輯 Grid 或複雜業務邏輯，就不適合用 Auto 引擎，應手寫 Core 類

## 相關連結

- [[Overview]]
- [[Core類]]（手寫頁面，不透過 Auto 引擎）
- [[Model 列表]]（與 Auto 引擎是不同系統）
