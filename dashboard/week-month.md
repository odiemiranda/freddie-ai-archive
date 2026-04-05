---
cssclasses: [dashboard-widget]
---
### This Week

`$= const w = dv.pages('"notes/weekly"').sort(p => p.file.name, "desc")[0]; dv.span(w ? w.file.link : "*No weekly note yet.*")`

### This Month

`$= const m = dv.pages('"notes/monthly"').sort(p => p.file.name, "desc")[0]; dv.span(m ? m.file.link : "*No monthly note yet.*")`
