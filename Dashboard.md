# Dashboard

## 最近 Dev Log

```dataview
TABLE date, project, summary
FROM "Projects"
WHERE type = "devlog"
SORT date DESC
LIMIT 20
```

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
