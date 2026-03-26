---
name: Work timezone PST/PDT
description: User works PST/PDT timezone Mon-Fri (8pm-4am Manila time). Penny should use PST for "today" during work week unless explicit date given.
type: feedback
agent: freddie
tags: [timezone, pst, pdt, manila, work, schedule, daily, penny, date]
---

Monday through Friday (or until user says otherwise), operate in PST/PDT Pacific timezone for daily notes. User's shift is 8pm-4am Manila time which maps to PST working hours. When the user says "today" at 10pm Manila on March 24, they mean March 23 PST.

**Why:** System timezone is UTC+8 (Manila) but work context is PST/PDT. Without this, Penny creates notes for the wrong date during work hours.

**How to apply:** Use PST/PDT as the working timezone for all date resolution in Penny tools unless the user explicitly provides a date. On weekends, default back to Manila time (or follow user's lead).
