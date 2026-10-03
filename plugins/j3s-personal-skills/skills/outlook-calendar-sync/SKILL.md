---
name: outlook-calendar-sync
description: >
  Reads Juhani's "Runtech RunPro Team" calendar from the Microsoft Outlook DESKTOP app (computer
  use, never web Outlook) and one-way syncs its events to the Google Calendar named "Runtech
  System". Snapshot-based, dedupes by summary + start date, and reports created/skipped counts.
  Use this skill whenever Juhani asks to sync the RunPro/work calendar to Google, asks for "vain
  synkka", asks to mirror his Outlook work calendar, or asks what's on the RunPro Team calendar.
  Never writes back to Outlook and never writes to any Google calendar other than "Runtech
  System". Does NOT triage mail or build a page; pair with outlook-mail-triage and
  erika-visual-page for those.
---

# Outlook calendar sync — RunPro Team → Google "Runtech System"

Keep Juhani's Google work calendar mirrored from the Outlook "Runtech RunPro Team" calendar. This
skill covers the calendar only. Mail triage lives in `outlook-mail-triage`; the HTML page lives in
`erika-visual-page`.

## Calendar memory (ledger) — read this first, every run

Canonical store: `SYSTEM/state/calendar.db` (SQLite; sibling of `email.db`). Schema in
`SYSTEM/engine/db/calendar_schema.sql`. Same sqlite rule as the Email DB: **runs on the Mac, not over the
sandbox mount** — in the sandbox `cp` to `/tmp`, edit, `cp` back.

Tables: `calendar_events` (event_key = summary|start_date; source, summary, start/end date, all_day,
start/end time, location, `google_synced`, `google_event_id`, status active|changed|vanished,
first_seen, last_seen, notes) and `sync_runs` (run log: window, created/skipped/changed/vanished
counts).

The sync behaviour below is unchanged — the ledger only gives it memory so it stops being blind
between runs:
- **Dedupe against the DB, not just a live Google read** — faster and survives across runs.
- **New:** event in Outlook, not in the ledger → create in Google, insert row with
  `google_event_id`, status `active`.
- **Changed:** event whose dates/time moved vs the ledger → flag `status='changed'` in the report;
  don't silently edit Google unless Juhani asks (then update and store the new values).
- **Vanished (cancellation):** a row seen on prior runs but **absent from today's Outlook read**
  → flag `status='vanished'` in the report as a likely cancellation. Do **not** auto-delete from
  Google — surface it and let Juhani confirm.
- Refresh `last_seen` on every event still present; write a `sync_runs` row at the end and `cp` the
  db back to the Mac.

The ledger starts empty; it populates on the first run. Do not fabricate calendar events to seed
it — only what's read from Outlook goes in.

## Hard rules

- Outlook **desktop app only**, via computer use. Never Outlook web, never IMAP.
- **Read-only in Outlook.** Never send, delete, move, or edit anything in Outlook.
- Google Calendar writes go **only** to the calendar named "Runtech System" — resolve its ID via
  `list_calendars` each run; never hardcode, never write to the primary calendar.
- The sync is one-way (Outlook → Google) and snapshot-based. Never write back to Outlook.
- Never invent event details. If a title or span is unreadable, click the event for the full text
  rather than guessing.

## Step 0 — access

Call `request_access` for "Microsoft Outlook", then `open_application`. If the Google Calendar
connector tools are deferred, load them via ToolSearch (`list_calendars`, `list_events`,
`create_event`).

## Step 1 — read the RunPro Team calendar

`cmd+2` → month view (Kuukausi). In the sidebar under "Omat kalenterit", ensure **only**
"Runtech RunPro Team" is checked — if other calendars are visible, their events must not leak
into the sync.

- Multi-day banners are all-day events; note exact start/end day from the column span — zoom on
  each week row to read titles and spans precisely.
- Timed events show a clock time in the cell (e.g. "14.00 …"); click them once to get the popover
  with full title, time range and location, then Escape.
- Truncated titles ("…") → click the event for the full text.
- Page forward with the chevron next to the month name; capture the current month plus the next
  one (≈6 weeks from today).

## Step 2 — sync to Google Calendar "Runtech System"

1. `list_calendars` → find "Runtech System", take its ID.
2. `list_events` on that calendar over the sync window (today → end of next month) — build the
   existing set.
3. **Dedupe** by summary + start date against the calendar ledger (`calendar_events`) and the
   Google read: skip events that already exist. Don't update or delete existing events unless
   asked; flag changed dates (`status='changed'`) and events that disappeared from Outlook
   (`status='vanished'`, likely cancellation) in the report instead of silently editing Google.
4. Create missing events:
   - All-day events: `allDay: true`, end date **exclusive** (last day + 1).
   - Timed events: exact times, `timeZone: Europe/Helsinki`, include location.
   - Description: `Synced from Outlook calendar: Runtech RunPro Team` (+ any useful note, e.g.
     "invite not yet answered in Outlook").
5. Verify with a final `list_events`. Update the ledger (insert new rows with `google_event_id`,
   refresh `last_seen`, mark changed/vanished), write a `sync_runs` row, `cp` the db back, and
   report created / skipped / changed / vanished counts.

## Handoff

- To triage work mail, run `outlook-mail-triage`.
- To visualise the calendar/briefing as an Erika-style HTML page, pass the content to
  `erika-visual-page`.

## Failure modes

- "Workspace still starting" from bash → wait and retry.
- Outlook window partly off-screen or covered: `open_application` brings it forward; don't
  resize unless necessary.
- A granted-app screenshot hides other apps — that's expected, not an error.
- If "Runtech System" calendar doesn't exist, stop and ask Juhani to create it in Google
  Calendar (the connector cannot create calendars).
