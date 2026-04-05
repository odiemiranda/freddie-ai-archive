---
cssclasses: [dashboard-widget]
---
### Today

```dataviewjs
const today = dv.date("today").toFormat("yyyy-MM-dd");
const page = dv.page("notes/daily/" + today);
if (page) {
  const tasks = page.file.tasks;
  const done = tasks.where(t => t.completed).length;
  dv.paragraph(`**Tasks:** ${done}/${tasks.length} completed`);
  const meetings = dv.pages('"notes/meetings"').where(p => p.file.name.startsWith(today));
  if (meetings.length > 0) {
    dv.paragraph("**Meetings:**");
    dv.list(meetings.map(m => m.file.link + (m.time ? " @ " + m.time : "")));
  } else {
    dv.paragraph("*No meetings today.*");
  }
} else {
  dv.paragraph("*No daily note yet.*");
}
```
