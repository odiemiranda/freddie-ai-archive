---
id: daily-{{date:YYYYMMDD}}
title: "{{date:YYYY-MM-DD}}"
tags: [daily]
connected: []
---

## Summary

| Metric | Count |
|--------|-------|
| Tasks completed | `$= const t = dv.current().file.tasks.where(t => t.section.subpath === "Tasks"); dv.span(t.where(t => t.completed).length + "/" + t.length)` |
| Sessions | `$= const tops = ["Summary","Tasks","Log","Session Work","Decisions","Learned","Meetings","Tomorrow","Notes"]; dv.span(new Set(dv.current().file.lists.where(l => !tops.includes(l.section.subpath) && !l.task).map(l => l.section.subpath).values).size)` |
| Decisions made | `$= dv.span(dv.current().file.lists.where(l => l.section.subpath === "Decisions" && !l.task).length)` |
| Things learned | `$= dv.span(dv.current().file.lists.where(l => l.section.subpath === "Learned" && !l.task).length)` |
| Items for tomorrow | `$= dv.span(dv.current().file.tasks.where(t => t.section.subpath === "Tomorrow").length)` |
| Meetings | `$= dv.span(dv.current().file.lists.where(l => l.section.subpath === "Meetings" && l.text.includes("[[")).length)` |
| Connected notes | `$= const c = dv.current().connected; dv.span(c ? c.length : 0)` |

## Tasks
- [ ]

## Log


## Notes

