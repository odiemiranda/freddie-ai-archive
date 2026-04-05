---
cssclasses: [dashboard-widget]
---
### Open Tasks

```dataview
TASK
FROM "notes/daily"
WHERE !completed
SORT file.name DESC
LIMIT 20
```
