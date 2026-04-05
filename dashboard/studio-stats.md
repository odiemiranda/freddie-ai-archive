---
cssclasses: [dashboard-widget]
---
### Studio

| | Count |
|--|-------|
| Track sets | `$= dv.span(dv.pages('"studio/tracks"').where(p => p.id && !p.id.includes("-v")).length)` |
| Track variations | `$= dv.span(dv.pages('"studio/tracks"').where(p => p.id && p.id.includes("-v")).length)` |
| Lyrics sets | `$= dv.span(dv.pages('"studio/lyrics"').where(p => p.id && !p.id.includes("-v")).length)` |
| Lyrics variations | `$= dv.span(dv.pages('"studio/lyrics"').where(p => p.id && p.id.includes("-v")).length)` |
