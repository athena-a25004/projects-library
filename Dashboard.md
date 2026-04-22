# Dashboard

## 最近 Dev Log

```dataview
TABLE date, project, summary
FROM "Projects" OR "Areas"
WHERE type = "devlog"
SORT date DESC
LIMIT 20
```

## 未完成的線頭

```dataview
TABLE file.link AS 筆記, project
FROM "Projects"
WHERE type = "devlog" AND contains(file.content, "## 明天待續")
SORT date DESC
LIMIT 10
```

---

## fai2 進行中專案

```dataview
TABLE repo, stack
FROM "Projects"
WHERE type = "project-overview" AND contains(tags, "fai2") AND status = "active"
SORT file.name ASC
```

## mf 進行中專案

```dataview
TABLE repo, stack
FROM "Projects"
WHERE type = "project-overview" AND contains(tags, "mf") AND status = "active"
SORT file.name ASC
```

## 所有專案

```dataview
TABLE repo, generation, status
FROM "Projects"
WHERE type = "project-overview"
SORT generation ASC, file.name ASC
```

---

## 模組筆記索引

```dataview
TABLE project, module
FROM "Projects"
WHERE type = "module-note"
SORT project ASC, module ASC
```
