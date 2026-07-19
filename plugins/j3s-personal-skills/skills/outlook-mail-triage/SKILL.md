---
name: outlook-mail-triage
description: >
  Juhani's work-mail triage from the Microsoft Outlook DESKTOP app (computer use, never web
  Outlook). Grows an Email DB each run: reads only mail newer than the last standpoint plus any
  unread/pinned, indexes it, and classifies everything with a traffic-light system (red = act
  now, yellow = monitor, green = info), producing a Finnish chat report of what needs his
  reaction. Use this skill whenever Juhani asks to read or triage his work email or Outlook, asks
  for a "liikennevaloraportti", "työpostit", "traffic light report", asks what needs his reaction,
  or asks "vain postit". Read-only in Outlook — never sends, replies, deletes, archives, moves, or
  marks anything. Does NOT touch the calendar or build a page; pair with outlook-calendar-sync and
  erika-visual-page for those.
---

# Outlook mail triage — Email DB + traffic-light report

Grow the Email DB from Outlook and produce a Finnish traffic-light report of Juhani's actionable
work mail. This skill covers mail only. Calendar sync lives in `outlook-calendar-sync`; the HTML
page lives in `erika-visual-page`.

## Email DB (memory) — read this first, every run

Canonical store: `PERSONAL ASSISTANT/DB/email.db` (SQLite; sibling of `balu.db`). Schema in
`DB/outlook_schema.sql`. Query helper: `python3 DB/email_query.py <cmd>` (`stats`, `red`,
`yellow`, `since <date>`, `waiting`, `search <term>`, `standpoint`). Human index:
`DB/email_index.md`.

**sqlite runs on the Mac, never over the sandbox mount** (mount raises "disk I/O error"). In the
sandbox: `cp DB/email.db /tmp/`, edit `/tmp/email.db`, then `cp` back. Native runs on J3s's Mac
are fine.

Tables: `emails` (msg_key = sha1 of sender|subject|received-date; folder, sender, subject,
received, addressed to/cc, `was_unread` = unread at first capture, pinned, replied, thread, gist,
ask, deadline, triage, status new|triaged|waiting_on|resolved, first_seen, last_seen, notes) and
`runs` (the standpoint log: run_at, high_water = max(received) seen, new/updated counts).

Each run:
1. **Read the standpoint:** `python3 DB/email_query.py standpoint` → the high_water timestamp.
2. **Read only what's new:** in Outlook, index every mail **newer than the standpoint**, plus any
   **unread** or **pinned** regardless of date (see Step 1). Don't re-read the whole archive.
3. **Index before triaging:** insert new rows (dedupe on msg_key — `INSERT OR IGNORE`), refresh
   `last_seen` on ones already present. Preserve `was_unread` from first capture.
4. **Accountability:** an item flagged red/yellow on a prior run that is still unresolved keeps
   its row — surface its age ("auki 3 pv") in the report. When Juhani says he answered one, set
   `status='waiting_on'` (see Step 2 watch list); when resolved, `status='resolved'`.
5. **Write the new standpoint:** after the pass, insert a `runs` row with the new high_water and
   counts, and `cp` the db back to the Mac.

Seeded 2026-07-18 by a full backfill (149 msgs, Action Items + Saapuneet, 2 Jun–18 Jul).

## Hard rules

- Outlook **desktop app only**, via computer use. Never Outlook web, never IMAP.
- **Read-only in Outlook.** Never send, reply, delete, archive, move, or mark anything. Clicking
  a message to read it is fine (it may mark as read — acceptable); changing anything else is not.
- Report language: Finnish. Paper machine terminology stays in English (tail threading, doctor
  blade, stabilizer, vacuum, etc.).
- Never invent facts. If a mail is ambiguous, quote what it says and flag the uncertainty.

## Step 0 — access

Call `request_access` for "Microsoft Outlook", then `open_application`.

## Step 1 — read the mail

Navigate: `cmd+1` = Mail, `cmd+2` = Calendar. The UI is **Finnish**: Saapuneet = Inbox,
Kiinnitetty = Pinned, Tänään = Today, Eilen = Yesterday, Tällä viikolla = This week,
Lähetetty = Sent.

Where the mail actually lives: the Inbox (Saapuneet) is normally **empty on both tabs**
(Tärkeät/Muu) — Juhani works at inbox zero. His working mail is in the **"Action Items"**
folder. Check Saapuneet first for strays, then do the real pass in Action Items. "Waiting On"
and "Read Later" exist but are out of scope unless he asks.

Coverage target (incremental): every **unread** mail, every **pinned** (Kiinnitetty) mail, and
everything **newer than the last-run standpoint** (from the Email DB) — not a fixed 3-day window.
After a gap (weekend, travel), the window stretches back to the standpoint automatically, so
nothing is missed. First run on an empty DB = full backfill of the folder.

Index from the **list view** (sender, subject, date, unread dot, pin/flag, reply-arrow, 2-line
preview) — do **not** open messages just to index them; opening marks them read and destroys the
`was_unread` signal the DB depends on. Open a message only when the ask/deadline isn't legible
from the list, and record that it became read.

Reading technique that works:
- Click a list item; read it in the reading pane. The pane text is legible in a full screenshot —
  zoom only for fine print.
- Clicking a conversation **expands** it in the list and shifts everything below; collapse it via
  the small chevron at the row's left edge before clicking the next item, or re-screenshot to get
  fresh coordinates. Never click stale coordinates after an expand/collapse.
- Quoted history inside one message often covers the whole thread — read the newest message fully
  before clicking older thread items.
- Scroll the list in small steps (4–6 ticks) and re-screenshot; date-section headers tell you when
  the 3-day window is exhausted.
- Note per mail: sender, subject, date/time, whether Juhani is in To or Cc, flags/pins,
  reply-arrow (= he already answered), and the concrete ask or deadline.

## Step 2 — traffic-light triage

Classify per Juhani's standing priorities: customer impact, delivery risk, technical blockers,
commitments, deadlines, unanswered messages.

- 🔴 **Punainen — act now**: a question or decision addressed to Juhani that is unanswered; an
  approval waiting on him (HR, expenses, resourcing); a customer-facing deadline at risk; anything
  where a customer will ask about it in the next call. Unread + directly addressed leans red.
- 🟡 **Keltainen — monitor/prepare**: open decisions where he already responded and waits on
  others; delivery risks owned by colleagues but on his projects; deadlines >1 week out that need
  preparation (e.g. PMDP reviews); unanswered meeting invites.
- 🟢 **Vihreä — info only**: threads colleagues own and are progressing; resolved threads;
  newsletters/surveys; internal R&D chatter where he is Cc.

Being in Cc lowers severity one notch unless the content names him or his team's resources.

Report format (chat message, Finnish):

```
## 🔴 Punainen — reagoi nyt
- **[Aihe]** (lähettäjä, aika): 1–2 lauseen tiivistys + mitä sinulta odotetaan
## 🟡 Keltainen — seuraa / valmistaudu
- ...
## 🟢 Vihreä — vain tiedoksi
- one-liners, koottuna
```

Order red items by urgency. Every red and yellow item names the owner and the concrete next step.
Show age for anything carried from a prior run ("auki 3 pv"). End with calendar observations if
they matter (overlapping site visits, unanswered invites, workshop closures colliding with trips)
— but only from what mail told you; reading the calendar itself is `outlook-calendar-sync`'s job.

### Waiting-on watch list (accountability)

When Juhani answers a red and now waits on someone else, set that row's `status='waiting_on'`
instead of dropping it. Each run, re-surface waiting_on items that have gone quiet past a
reasonable window (`python3 DB/email_query.py waiting`) under a **"⏳ Odottaa vastausta"** section
so commitments don't die in silence. Mark `status='resolved'` once closed.

**Concur is out of scope** — expense/HR approvals are handled by Juhani by hand; don't try to read
or action them, and don't invent Concur items.

## Handoff

- To visualise this report as an Erika-style HTML page, pass the report content to
  `erika-visual-page`.
- To read and sync the RunPro Team calendar, run `outlook-calendar-sync`.

## Failure modes

- "Workspace still starting" from bash → wait and retry.
- Outlook window partly off-screen or covered: `open_application` brings it forward; don't
  resize unless necessary.
- A granted-app screenshot hides other apps — that's expected, not an error.
- If the mail pass finds nothing red, say so plainly — don't inflate yellows into reds.
