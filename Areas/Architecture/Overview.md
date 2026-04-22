---
type: area-note
area: Architecture
tags: [architecture]
---

# 架構總覽

各系統的架構決策、設計模式與技術選型紀錄。

## 世代區分

| 世代 | 路徑 | API 路徑 | 主要技術 |
|------|------|----------|----------|
| mf（舊） | `~/mf` | bac-apis → MainFramework → Backend | Vue 2, Element UI |
| fai2（新） | `~/fai2` | 各專案內建 NestJS 轉拋層 → Backend | Nuxt 4, Vue 3, NestJS |

## 系統關聯圖

```
平台層    eip（入口）+ IT（權限/設定）
               ↓ 提供登入/權限
業務核心  pms（飯店）  fbm（餐飲）  [work-flow]（簽核+人事）
               ↓              ↓
整合層         crs（預約中心，聚合 pms 訂房 + fbm 訂席/初洽）
報表層    bi（跨系統報表）
共用庫    bac-apis, vue-bac-lib-v-3, nuxt-base, nuxt-navigator
```

## 架構決策紀錄（ADR）

<!-- 每個重大架構決策放一個 H3，格式如下 -->

### ADR-001 fai2 採用 NestJS 轉拋層

- **狀態**：已採用
- **背景**：
- **決策**：
- **後果**：

## 相關筆記

- [[Code Standards/Overview]]
