---
type: resource
resource: Nuxt 4
tags: [nuxt4, nuxt, frontend]
---

# Nuxt 4 快速參照

## 專案結構（fai2 慣例）

```
app/
├── components/       # 自動引入的元件
├── composables/      # 自動引入的 composable
├── layouts/
├── pages/            # 路由
├── plugins/
└── utils/
server/
├── api/              # NestJS 轉拋層
└── middleware/
nuxt.config.ts
```

## 路由規則

| 目錄結構 | 產生路由 |
|---------|---------|
| `pages/index.vue` | `/` |
| `pages/users/index.vue` | `/users` |
| `pages/users/[id].vue` | `/users/:id` |
| `pages/[...slug].vue` | 萬用路由 |

## Nuxt 4 vs Nuxt 2/3 差異

<!-- 記錄升級或遷移時遇到的差異 -->

## 常見問題

<!-- 開發過程中遇到的問題與解法 -->

## server/ NestJS 轉拋層

```ts
// server/api/reservation.post.ts
export default defineEventHandler(async (event) => {
  const body = await readBody(event)
  // 轉拋到 Backend
  return $fetch('http://backend/api/...', { method: 'POST', body })
})
```

## 相關連結

- [[Vue3/Vue3]]
- [[../Projects/[shared-libs]/nuxt-base]]
- [[../Projects/[shared-libs]/nuxt-navigator]]
