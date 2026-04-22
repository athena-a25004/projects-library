---
type: module-note
project: vue-bac-lib-v-3
module: Core 類
tags: [vue-bac-lib, core, architecture]
---

# Core 類

15 個 Core class，提供頁面骨架與狀態管理。業務專案繼承後只需關注業務邏輯，不直接操作 Syncfusion API。

## 繼承關係

```
Core（基礎，context 注入、共用方法）
│
├── CoreField（欄位定義管理）
│   └── CoreGridField（Grid 欄位，extends CoreField）
│
├── CoreSetup（設定頁面）
│
├── CoreSearch（搜尋條件管理）
│   └── CoreSearchAndGrid（搜尋 + Grid 組合，extends CoreSearch）
│   └── CoreSearchAndPivot（搜尋 + PivotView 組合，extends CoreSearch）
│
├── CoreCURD（新增/修改/刪除操作）
│
├── CoreDialog（對話框管理）
│
├── CoreToolbar（Toolbar 按鈕管理）
│
├── CoreTab（分頁管理）
│
├── CorePivot（PivotView 功能）
│
├── CoreReport（報表功能）
│
├── CoreDashboard（Dashboard 功能）
│
└── CoreHandWrite（手寫簽名）
```

> 實際繼承細節以 `src/cores/` 原始碼為準，上圖為概念關係。

## 各類說明

### `Core` — 基礎類
`src/cores/Core.ts`

所有 Core 類的根，提供：
- context 注入（語言、權限、全局設定）
- 共用工具方法
- 初始化流程（`setup()` → `afterSetup()`）

業務使用：幾乎不直接繼承，多透過子類使用。

---

### `CoreField` — 欄位定義
`src/cores/CoreField.ts`

管理表單/搜尋欄位的 Field 物件陣列。

```ts
class MyPage extends CoreField {
  fields = [
    new BacTextBox({ label: '名稱', field: 'name' }),
    new BacDropdownlist({ label: '狀態', field: 'status', dataSource: [...] }),
  ]
}
```

---

### `CoreGridField` — Grid 欄位
`src/cores/CoreGridField.ts`

extends `CoreField`，額外管理 Grid 的 column 定義。

```ts
class MyPage extends CoreGridField {
  gridColumns = [
    new GridTextBox({ field: 'name', headerText: '名稱' }),
    new GridDropdownList({ field: 'status', headerText: '狀態', dataSource: [...] }),
  ]
}
```

---

### `CoreSearch` — 搜尋條件
`src/cores/CoreSearch.ts`

管理搜尋條件的組合與查詢觸發。

---

### `CoreSearchAndGrid` — 搜尋 + Grid
`src/cores/CoreSearchAndGrid.ts`

最常用的 Core 類之一。整合搜尋條件 + Grid 呈現，內建分頁、排序、匯出。

```ts
class MyPage extends CoreSearchAndGrid {
  // 定義搜尋欄位
  searchFields = [...]
  // 定義 Grid 欄位
  gridColumns = [...]
  // 查詢 API 呼叫
  async fetchData(condition) { return await api.list(condition) }
}
```

---

### `CoreSearchAndPivot` — 搜尋 + PivotView
`src/cores/CoreSearchAndPivot.ts`

整合搜尋條件 + Syncfusion PivotView。

---

### `CoreCURD` — 新增/修改/刪除
`src/cores/CoreCURD.ts`

提供標準 CURD 操作流程（驗證 → API → 通知）。

---

### `CoreDialog` — 對話框
`src/cores/CoreDialog.ts`

管理頁面內對話框的開關與資料傳遞。

---

### `CoreToolbar` — Toolbar 按鈕
`src/cores/CoreToolbar.ts`

動態定義 Toolbar 按鈕（新增、匯出、自訂按鈕），搭配 [[Model 列表#toolbar]] 使用。

---

### `CoreTab` — 分頁
`src/cores/CoreTab.ts`

管理多 tab 頁面的切換狀態。

---

### `CoreSetup` — 設定頁
`src/cores/CoreSetup.ts`

系統設定類頁面（如參數設定、基本資料）。

---

### `CorePivot` — PivotView
`src/cores/CorePivot.ts`

單獨使用 Syncfusion PivotView 的核心類。

---

### `CoreReport` — 報表
`src/cores/CoreReport.ts`

<!-- 待補充 -->

---

### `CoreDashboard` — Dashboard
`src/cores/CoreDashboard.ts`

整合 Syncfusion Dashboard Layout（圖表、KPI 卡片）。

---

### `CoreHandWrite` — 手寫簽名
`src/cores/CoreHandWrite.ts`

手寫簽名功能，對應 UI 元件 `BacImage`。

---

## 使用模式

### 基本繼承

```ts
// MyFeature.ts（業務 Core）
import { CoreSearchAndGrid } from 'vue-bac-lib'

export class MyFeature extends CoreSearchAndGrid {
  // 宣告欄位、實作 fetchData
}
```

```vue
<!-- MyFeature.vue -->
<script setup lang="ts">
import { MyFeature } from './MyFeature'
const core = new MyFeature()
await core.setup()
</script>

<template>
  <!-- 使用 core 的 Model 渲染 -->
  <BRenderer :core="core" />
</template>
```

## 相關連結

- [[Overview]]
- [[Model 列表]]（Core 類通常搭配 Model 使用）
- [[Auto引擎]]（BRenderer 接收 Core 實例）
