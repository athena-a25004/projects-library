---
type: module-note
project: vue-bac-lib-v-3
module: Utils & Services
tags: [vue-bac-lib, utils, services, i18n]
---

# Utils & Services

## utils/ 工具函數

### `useTools()` — 核心工具 Hook

```ts
import { useTools } from 'vue-bac-lib'
const tools = useTools()
```

<!-- 待補充：useTools 提供哪些方法 -->

---

### `useDialog()` — 全局對話框

`src/utils/global-dialogs.ts`

```ts
import { useDialog } from 'vue-bac-lib'

const dialog = useDialog()
dialog.open('dialogName', data)     // 開啟對話框並傳入資料
dialog.close('dialogName')          // 關閉
```

---

### `delayWrapper()` — 延遲包裝

`src/utils/delayWrapper.ts`

將同步操作包裝成延遲執行，用於處理 Syncfusion 元件需要在 DOM 更新後才能呼叫的場景。

```ts
import { delayWrapper } from 'vue-bac-lib'
delayWrapper(() => {
  gridInstance.refresh()
}, 100)
```

---

### i18n 工具

`src/utils/i18n.ts`

```ts
import { setSystemEnabledLocale } from 'vue-bac-lib'

// 設定系統支援的語系
setSystemEnabledLocale(['zh-TW', 'en-US'])
```

---

### `dynamicComponentLoader` — 動態元件載入

`src/utils/dynamicComponentLoader.ts`

<!-- 待補充 -->

---

### `ej2Template` — EJ2 模板工具

`src/utils/ej2Template.ts`

幫助在 Syncfusion 元件中使用 Vue 元件作為 Template（如 Grid column template）。

---

## services.ts — 全局服務

`src/services.ts`（約 600 行）

### 權限注入

```ts
import { injectPermissions } from 'vue-bac-lib'

// 在 app 初始化時注入權限資料
injectPermissions({
  canCreate: true,
  canEdit: true,
  canDelete: false,
})
```

---

### 數字格式化

```ts
import { setDecimals, getDecimals, numberDisplayFormat } from 'vue-bac-lib'

setDecimals(2)         // 設定全局小數位
const dec = getDecimals()
numberDisplayFormat(1234567.89)  // → '1,234,567.89'
```

---

### 條件組合

```ts
import { combineCondition } from 'vue-bac-lib'

const query = combineCondition([
  { field: 'status', value: '1', operator: 'equal' },
  { field: 'date', value: '2026-01-01', operator: 'greaterThan' },
])
```

---

### 搜尋條件轉查詢

```ts
import { searchConditionToQuery } from 'vue-bac-lib'

const query = searchConditionToQuery(searchFields)
```

---

### 資料驗證工具

```ts
import { isDataContainer, isValidationField, isDataChange } from 'vue-bac-lib'

isDataContainer(value)       // 是否為資料容器
isValidationField(field)     // 是否為需驗證的欄位
isDataChange(original, current)  // 資料是否有變更
```

---

### 其他工具

```ts
import { excludeNull, base64ToArrayBuffer, setTestMode } from 'vue-bac-lib'

excludeNull(obj)              // 移除物件中的 null 值
base64ToArrayBuffer(base64)   // Base64 轉 ArrayBuffer（PDF 顯示用）
setTestMode(true)             // 開啟測試模式（dev 用）
```

---

## context.ts — 全局 Context

`src/context.ts`（約 600 行）

提供跨元件的全局狀態，由 `install.ts` 在 Vue Plugin 安裝時初始化。

<!-- 待補充：context 提供哪些具體的狀態與方法 -->

---

## instance.ts — 實例管理

`src/instance.ts`

管理 Syncfusion 元件實例的生命週期（取得、儲存、清除）。

<!-- 待補充 -->

---

## 相關連結

- [[Overview]]
- [[Core類]]（Core 類內部大量使用這些工具）
