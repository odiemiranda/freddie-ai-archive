---
cssclasses:
  - dashboard-widget
---
### Recent Meetings

```dataview
TABLE time AS "Time", attendees AS "With"
FROM "notes/meetings"
SORT file.name DESC
LIMIT 5
```
