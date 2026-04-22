---
type: resource
resource: Syncfusion
tags: [syncfusion, ui, components]
---

# Syncfusion 快速參照

## 使用於哪些專案

| 專案 | 用到的 Syncfusion 元件 |
|------|----------------------|
| fbm | Image Editor（座位圖編輯） |
| crs | Diagrams（預約流程圖） |

## 安裝

```bash
npm install @syncfusion/ej2-vue-...
```

> 需要授權 key，放在 `nuxt.config.ts` 或 `.env`。

## Image Editor（fbm 座位圖）

```vue
<script setup>
import { ImageEditorComponent } from '@syncfusion/ej2-vue-image-editor'
</script>

<template>
  <ejs-imageeditor />
</template>
```

### 常用事件

| 事件 | 說明 |
|------|------|
| `created` | 元件初始化完成 |
| `toolbarItemClicked` | 工具列點擊 |

## Diagrams（crs 預約流程）

<!-- 待補充 -->

## 已知問題 / 坑

<!-- 開發過程中踩到的 Syncfusion 問題記在這裡 -->
