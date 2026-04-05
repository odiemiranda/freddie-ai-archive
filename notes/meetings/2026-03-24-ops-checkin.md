---
id: meeting-20260324-065038
title: Ops - Checkin
tags: [meeting]
date: 2026-03-24
daily: "[[notes/daily/2026-03-24]]"
attendees: [Odelon, Farez, Victor, Adrianus, Nicky, Mike Zachar, Erica Serna]
connected: []
time: "11:15 PM"
---

# Ops - Checkin

## Agenda

- Team status updates and blockers

## Decisions

- Investigate locking AirTable view to prevent unauthorized field name changes causing system failures (raised by Mike, Erica, Odelon)
- Erica to schedule separate call re: IP/VPN flagging from concurrent testing
- New process: track Tixie issues in separate cards so Res and Odelon can pick up tickets, relieving Victor as sole owner
- VPN rotation recommended: reconnect every hour to cycle IP address

## Action Items

- [ ] **Odelon** — Submit PR for 60-min token refresh cycle (OT-1598 Tixie auth)
- [ ] **Farez** — Unblock e2e testing for OT-1471 (headless base update) — "phone number not found" error
- [ ] **Victor** — Discuss broker logic before deploying service account API; continue Tixie white screen fix
- [ ] **Adrianus** — Investigate match cancellation tool (incorrectly canceling 'know' items, ignoring actual targets); debug error provincial extension AirTable errors; review Victor's Tixie releases
- [ ] **Nicky** — Update Lucid diagram; compile price point/Konami mismatch requests for Rubseen; investigate if "we tool" is causing tag deletions; follow up on Slack fulfillment report
- [ ] **Mike** — Troubleshoot purchasing accounts cron job timing out at 9 min; build Terraform setup to migrate free speech system from Firebase/Firestorm to AWS
- [ ] **Erica** — Schedule IP/VPN call; set up separate Tixie issue cards for team distribution
- [ ] **Team** — Investigate locking AirTable view to prevent unauthorized field changes

## Notes

### Odelon Miranda
- Completed API development for OT-1598: challenge, get token, refresh token endpoints — all tested
- Implementing 60-min token refresh cycle for Tixie, PR expected today
- Suggested Tixie white screen is Ticketmaster flagging IP due to multiple accounts from same location
- Recommended VPN with hourly reconnect to cycle IP address

### Farez Ramilo
- Blocked on e2e testing for OT-1471 (headless base update) — persistent "phone number not found" error
- Completed and pushed PR for Daily checkout check report (OT-1604) for Nicky — root cause was AirTable query returning zero results

### Victor Panisa
- Removed city detection for canceled events
- Added extra logging for address form filling issue (cannot reproduce locally)
- Implemented temporary overlay introduction bypass for his team (Nicky, Ray, Adrianus, himself)
- Working on service account API — troubleshooting split source of truth, needs to discuss broker logic before deploying
- Tixie white screen: Ticketmaster rejecting session after "Phantom capture" check fails
- Confirmed interest in continuing service account and related API work

### Adrianus Bussem
- Will investigate match cancellation tools — incorrectly canceling items set to 'know', ignoring those that should be canceled
- Will debug error provincial extension — "bad request" and "forbidden" errors connecting to AirTable
- Will review Victor's Tixie releases

### Nicky Diotay
- Updating Lucid diagram and compiling price point/Konami mismatch requests for Rubseen
- Tags still being deleted — investigating if "we tool" is the cause
- Confirmed urgent fulfillment report stopped sending to Slack due to zero-result AirTable query (same root cause as OT-1604)
- Reported persistent Tixie issues: checkout autofill not working, white screen after login

### System and AirTable Concerns
- Mike, Erica, and Odelon frustrated by unauthorized AirTable field name changes causing system failures
- Discussion on locking the relevant view
- Mike troubleshooting purchasing accounts cron job timing out at 9 minutes
- Mike building new Terraform setup to migrate free speech system from Firebase/Firestorm to AWS

### Tixie IP/VPN Troubleshooting
- Erica scheduled separate call — suspects concurrent testing with Raelynn from same house is flagging IP despite dedicated IP VPN
- Giving Raelynn a new device improved her success rate
- Odelon recommended hourly VPN reconnect to rotate IP

### Team and Personnel
- Erica announced new process: Tixie issues tracked in separate cards so Res and Odelon can pick up tickets, relieving Victor as sole owner
- Mike reminded team: today is Mitches' last day, farewell event planned
