---
cssclasses: [dashboard-widget]
---
### Vault

| | Count |
|--|-------|
| Daily notes | `$= dv.span(dv.pages('"notes/daily"').length)` |
| Weekly | `$= dv.span(dv.pages('"notes/weekly"').length)` |
| Monthly | `$= dv.span(dv.pages('"notes/monthly"').length)` |
| Meetings | `$= dv.span(dv.pages('"notes/meetings"').length)` |
| Inbox | `$= dv.span(dv.pages('"notes/inbox"').length)` |
