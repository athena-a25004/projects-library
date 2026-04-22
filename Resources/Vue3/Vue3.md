---
type: resource
resource: Vue 3
tags: [vue3, frontend]
---

# Vue 3 快速參照

## Composition API 常用模式

### `<script setup>` 基本結構

```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

const props = defineProps<{
  id: number
}>()

const emit = defineEmits<{
  change: [value: string]
}>()

const data = ref<string>('')
const derived = computed(() => data.value.trim())

onMounted(() => {
  // init
})
</script>
```

### Composable 模式

```ts
// useXxx.ts
export function useXxx() {
  const state = ref(...)

  function doSomething() { ... }

  return { state, doSomething }
}
```

## 常用 API 速查

| API | 用途 |
|-----|------|
| `ref()` | 基本型別響應式 |
| `reactive()` | 物件響應式 |
| `computed()` | 衍生值 |
| `watch()` | 副作用監聽 |
| `watchEffect()` | 自動追蹤依賴的副作用 |
| `provide/inject` | 跨層注入 |

## 實用技巧

<!-- 開發過程中遇到的 Vue 3 技巧記在這裡 -->

## 相關連結

- [[Nuxt4/Nuxt4]]
- [[../Areas/Code Standards/Vue 元件規範]]
