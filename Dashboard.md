---
id: dashboard
title: Dashboard
tags: [dashboard]
cssclasses: [dashboard]
---

# `= dateformat(date(today), "DDDD")`

`$= dv.span("[[notes/daily/" + dv.date("today").toFormat("yyyy-MM-dd") + "|Today's Note]]")` | `$= dv.span("[[notes/daily/" + dv.date("today").minus({days:1}).toFormat("yyyy-MM-dd") + "|Yesterday]]")` | `$= const w = dv.pages('"notes/weekly"').sort(p => p.file.name, "desc")[0]; dv.span(w ? w.file.link : "")` | `$= const m = dv.pages('"notes/monthly"').sort(p => p.file.name, "desc")[0]; dv.span(m ? m.file.link : "")`

---

> [!multi-column]
>
>> [!todo]+ Today
>> ```dataviewjs
>> const today = dv.date("today").toFormat("yyyy-MM-dd");
>> const page = dv.page("notes/daily/" + today);
>> if (page) {
>>   const tasks = page.file.tasks;
>>   const done = tasks.where(t => t.completed).length;
>>   dv.paragraph(`**Tasks:** ${done}/${tasks.length} completed`);
>>   const meetings = dv.pages('"notes/meetings"').where(p => p.file.name.startsWith(today));
>>   if (meetings.length > 0) {
>>     dv.paragraph("**Meetings:**");
>>     dv.list(meetings.map(m => m.file.link + (m.time ? " @ " + m.time : "")));
>>   } else {
>>     dv.paragraph("*No meetings today.*");
>>   }
>> } else {
>>   dv.paragraph("*No daily note yet.*");
>> }
>> ```
>
>> [!summary]+ Vault
>> | | Count |
>> |--|-------|
>> | Daily notes | `$= dv.span(dv.pages('"notes/daily"').length)` |
>> | Weekly | `$= dv.span(dv.pages('"notes/weekly"').length)` |
>> | Monthly | `$= dv.span(dv.pages('"notes/monthly"').length)` |
>> | Meetings | `$= dv.span(dv.pages('"notes/meetings"').length)` |
>> | Inbox | `$= dv.span(dv.pages('"notes/inbox"').length)` |
>
>> [!example]+ Studio
>> | | Count |
>> |--|-------|
>> | Track sets | `$= dv.span(dv.pages('"studio/tracks"').where(p => p.id && !p.id.includes("-v")).length)` |
>> | Variations | `$= dv.span(dv.pages('"studio/tracks"').where(p => p.id && p.id.includes("-v")).length)` |
>> | Lyrics sets | `$= dv.span(dv.pages('"studio/lyrics"').where(p => p.id && !p.id.includes("-v")).length)` |
>> | Variations | `$= dv.span(dv.pages('"studio/lyrics"').where(p => p.id && p.id.includes("-v")).length)` |

> [!multi-column]
>
>> [!attention]+ Open Tasks
>> ```dataview
>> TASK
>> FROM "notes/daily"
>> WHERE !completed
>> SORT file.name DESC
>> LIMIT 15
>> ```
>
>> [!help]+ Recent Meetings
>> ```dataview
>> TABLE time AS "Time", attendees AS "With"
>> FROM "notes/meetings"
>> SORT file.name DESC
>> LIMIT 5
>> ```

> [!multi-column]
>
>> [!info]+ Recent Tracks
>> ```dataview
>> TABLE title AS "Title", dateformat(file.ctime, "MMM d") AS "Date"
>> FROM "studio/tracks"
>> WHERE id AND !contains(id, "-v")
>> SORT file.ctime DESC
>> LIMIT 8
>> ```
>
>> [!quote]+ Recent Lyrics
>> ```dataview
>> TABLE title AS "Title", dateformat(file.ctime, "MMM d") AS "Date"
>> FROM "studio/lyrics"
>> WHERE id AND !contains(id, "-v")
>> SORT file.ctime DESC
>> LIMIT 8
>> ```
