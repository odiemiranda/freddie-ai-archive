---
cssclasses: [dashboard-widget]
---
### Recent Lyrics

```dataview
TABLE title AS "Title", dateformat(file.ctime, "MMM d") AS "Date"
FROM "studio/lyrics"
WHERE id AND !contains(id, "-v")
SORT file.ctime DESC
LIMIT 10
```
