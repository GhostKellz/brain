---
type: meta
title: "Dashboard"
created: 2026-06-21
updated: 2026-06-21
tags:
  - meta
  - dashboard
status: stable
related:
  - "[[Wiki Index]]"
---

# Wiki Dashboard

Live views over the vault. Requires the Dataview plugin.

## Recent Activity
```dataview
TABLE type, status, updated FROM "wiki" SORT updated DESC LIMIT 15
```

## Seed Pages (Need Development)
```dataview
LIST FROM "wiki" WHERE status = "seed" SORT updated ASC
```

## Developing Pages
```dataview
LIST FROM "wiki" WHERE status = "developing" SORT updated ASC
```

## Entities Missing Sources
```dataview
LIST FROM "wiki/entities" WHERE !sources OR length(sources) = 0
```

## Page Counts by Type
```dataview
TABLE length(rows) AS Pages FROM "wiki" GROUP BY type SORT length(rows) DESC
```
