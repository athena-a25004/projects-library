# Athena Project Library

開發專案知識庫，基於 PARA 變體架構管理跨專案文件。

---

## 資料夾結構

```
athena-proj-library/
├── Projects/
│   ├── [platform]/        eip（入口）, IT（基礎設定）, MainFramework, template
│   ├── [shared-libs]/     bac-apis, vue-bac-lib-v-3, nuxt-base, nuxt-navigator
│   ├── [work-flow]/       flow2（簽核）, hr2（人事）← 從 eip 拆離，互相高度相依
│   ├── bi/                報表系統
│   ├── crs/               預約中心（整合 pms 訂房 + fbm 訂席/初洽）
│   ├── fbm/               餐飲系統（Food and Beverage Management）
│   │   ├── Overview.md
│   │   ├── 訂位.md
│   │   ├── 訂席.md
│   │   └── 初洽.md
│   └── pms/               飯店前台系統
├── Areas/
│   ├── Architecture/
│   ├── DevOps/
│   └── Code Standards/
├── Resources/
│   ├── Vue3/
│   ├── Nuxt4/
│   └── Syncfusion/
├── Archive/
├── Templates/
│   ├── Dev Log.md
│   └── Project Overview.md
└── Dashboard.md           Dataview 聚合視圖
```

### 命名規則
- `[分類名]/` — 資料夾是分類，非專案
- `專案名/` — 資料夾是單一專案

---

## 系統架構概覽

### 世代區分

| 世代 | 路徑 | API 路徑 | 主要技術 |
|------|------|----------|----------|
| mf（舊） | `~/mf` | bac-apis → MainFramework → Backend | Vue 2, Element UI |
| fai2（新） | `~/fai2` | 各專案內建 NestJS 轉拋層 → Backend | Nuxt 4, Vue 3, NestJS |

### 系統關聯

```
平台層    eip（入口）+ IT（權限/設定）
               ↓ 提供登入/權限
業務核心  pms（飯店）  fbm（餐飲）  [work-flow]（簽核+人事）
               ↓              ↓
整合層         crs（預約中心，聚合 pms 訂房 + fbm 訂席/初洽）
報表層    bi（跨系統報表）
共用庫    bac-apis, vue-bac-lib-v-3, nuxt-base, nuxt-navigator
```

---

## 筆記規劃

### 每個專案的文件結構（top-down）
1. `Overview.md` — 目標、架構摘要、重要決策、技術債
2. 業務模組筆記（如 fbm 的訂位/訂席/初洽）— 業務邏輯、資料結構
3. `Dev Log/` — 每日開發紀錄（含 frontmatter 供 Dataview 聚合）
4. `_ai-context.md` — 給 AI 的密集摘要（最後產）

### Dev Log frontmatter 格式
```yaml
---
type: devlog
date: YYYY-MM-DD
project: 專案名
summary: 一行摘要
tags: []
---
```

### 開發時值得記錄的四類
1. **決策** — 為什麼這樣做、刻意跳過什麼
2. **卡關與解法** — 問題關鍵字 + 解法
3. **意外發現** — 預期之外的行為或邏輯
4. **未完成的線頭** — TODO、下次接續點

---

## 版號與文件對齊規則

### 分支模型

| 分支 | 版號 | 說明 |
|------|------|------|
| master | n | 客戶的穩定版本 |
| release | n+1 | 已發布、尚未落地的版本 |
| develop | n+2 | 開發中版本，新功能以此為基礎 |

### 核心原則

1. **文件永遠對齊 develop（n+2）** — 開發者用，查閱頻率最高
2. **不依版號分文件** — 只維護一份，記現狀 + 記變更
3. **差異用「尚未落地的變更」追蹤** — 兩層 buffer 對應 release / master 的落差
4. **相依性變更雙向記錄** — A 改了 B 也要寫，用 `[[連結#heading]]` 互指

### Overview.md 版號欄位

```yaml
---
version: x.y.z           # develop 版號（文件描述的版本）
version-release: x.y.z   # release 分支版號
version-master: x.y.z    # master 分支版號
---
```

### 版本推進時的更新流程

- **release → master 落地時**：`version-master` 更新，清空「在 release 但尚未到 master」
- **develop → release 時**：`version-release` 更新，開發中項目搬到 release 區塊

### Breaking Changes 格式

```markdown
### vX.Y.Z — 單句總結變更
- **日期**：YYYY-MM-DD
- **影響**：A ↔ B
- **before**：舊行為
- **after**：新行為
- **原因**：為什麼要改
```

---

## 插件需求
- **Dataview** — Dashboard 聚合、Dev Log 時間線查詢
- **Templater**（建議）— 套用 Templates/ 內的模板
