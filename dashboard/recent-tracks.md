---
cssclasses: [dashboard-widget]
---
### Recent Tracks

```dataview
TABLE title AS "Title", dateformat(file.ctime, "MMM d") AS "Date"
FROM "studio/tracks"
WHERE id AND !contains(id, "-v")
SORT file.ctime DESC
LIMIT 10
```
