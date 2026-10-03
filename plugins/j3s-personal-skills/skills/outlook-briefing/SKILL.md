---
name: outlook-briefing
description: >
  Juhani's work-mail and calendar briefing from the Microsoft Outlook DESKTOP app (computer use,
  never web Outlook). Reads unread, pinned and last-3-days mail, classifies everything with a
  traffic-light system (red = act now, yellow = monitor, green = info), reads the "Runtech RunPro
  Team" Outlook calendar, syncs its events to the Google Calendar "Runtech System", and renders
  the briefing as a minimalist Erika-style HTML page saved to OUTPUTS. Use this skill whenever Juhani asks to read or
  triage his work email or Outlook, asks for a "liikennevaloraportti", "työpostit", "traffic light
  report", "outlook briefing", asks what needs his reaction, or asks to sync the RunPro/work
  calendar to Google — even if he only asks for one part (mail only, sync only, page only). Partial
  runs are supported.
---

# Outlook briefing — mail triage + calendar sync + visual page

Produce a Finnish traffic-light report of Juhani's actionable work mail, keep his Google work
calendar mirrored from Outlook, and render the result as a polished HTML page.

A **full run executes all five steps** — the HTML page (step 5) is part of the deliverable, not
an extra. Skip steps only on an explicit partial request: "vain postit" → steps 1–2;
"vain synkka" → steps 3–4; "tee siitä sivu" / "ei sivua" → only/except step 5 (a page-only run
reuses the most recent report in the conversation).

## Execution topology — a diamond, not a chain

This skill has been read as a five-step chain. It isn't one. **The mail branch and the calendar
branch never read each other's output** — triage does not consume calendar events, and the sync does
not consume the mail. Only the page needs both.

```
              ┌─ 1 read mail  → 2 triage        ─┐
   start ─────┤                                  ├──→ 5 page (merge, one owner)
              └─ 3 read calendar → 4 sync Google ─┘
```

**Real edges:** 1→2, 3→4, and both branches → 5. **Fake edge:** 2→3. Never wait for the triage to
finish before touching the calendar because the numbering suggests it.

**But: the Outlook desktop GUI is an exclusive resource.** Steps 1 and 3 both drive the same window
through computer use, so they *cannot* physically overlap — one mouse, one front app. Be honest
about this rather than pretending to parallelise: the win in this variant is that step **4** (Google
Calendar connector, no GUI) and step **2** (classification of already-captured mail, no GUI) are
pure reasoning/API work that must not block the GUI branch.

Practical order for the computer-use variant:
1. Take the GUI once: capture the mail (step 1), then switch `cmd+2` and capture the calendar
   (step 3). **One pass at the machine, both captures done, then release it.**
2. Off the GUI, run triage (step 2) and the Google sync (step 4) as independent work — issue their
   tool calls in the same block where the runtime allows it.
3. Merge into the page (step 5), one owner, only once both have returned.

For a genuinely parallel run with no GUI contention, use **`outlook-briefing-mcp`**, which reads
Outlook through the local `mac-outlook` MCP server and can fan out both reads at once.

**One writer per file.** The mail branch and the calendar branch never write the same artifact; only
the merge (step 5) writes the HTML page in `OUTPUTS/`.

**Partial runs** are subgraphs: "vain postit" = the top branch only, "vain synkka" = the bottom
branch only. Neither needs the other to complete.

## Hard rules

- Outlook **desktop app only**, via computer use. Never Outlook web, never IMAP.
- **Read-only in Outlook.** Never send, reply, delete, archive, move, or mark anything. Clicking
  a message to read it is fine (it may mark as read — acceptable); changing anything else is not.
- Google Calendar writes go **only** to the calendar named "Runtech System" — resolve its ID via
  `list_calendars` each run; never hardcode, never write to the primary calendar.
- Report language: Finnish. Paper machine terminology stays in English (tail threading, doctor
  blade, stabilizer, vacuum, etc.).
- Never invent facts. If a mail is ambiguous, quote what it says and flag the uncertainty.

## Step 0 — access

Call `request_access` for "Microsoft Outlook", then `open_application`. If the Google Calendar
connector tools are deferred, load them via ToolSearch (`list_calendars`, `list_events`,
`create_event`).

## Step 1 — read the mail

Navigate: `cmd+1` = Mail, `cmd+2` = Calendar. The UI is **Finnish**: Saapuneet = Inbox,
Kiinnitetty = Pinned, Tänään = Today, Eilen = Yesterday, Tällä viikolla = This week,
Lähetetty = Sent.

Where the mail actually lives: the Inbox (Saapuneet) is normally **empty on both tabs**
(Tärkeät/Muu) — Juhani works at inbox zero. His working mail is in the **"Action Items"**
folder. Check Saapuneet first for strays, then do the real pass in Action Items. "Waiting On"
and "Read Later" exist but are out of scope unless he asks.

Coverage target: every **unread** mail, every **pinned** (Kiinnitetty) mail, and everything from
the **last 3 days** (Tänään + Eilen + the matching dates under Tällä viikolla).

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
End with calendar observations if they matter (overlapping site visits, unanswered invites,
workshop closures colliding with trips).

## Step 3 — read the RunPro Team calendar

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

## Step 4 — sync to Google Calendar "Runtech System"

1. `list_calendars` → find "Runtech System", take its ID.
2. `list_events` on that calendar over the sync window (today → end of next month) — build the
   existing set.
3. **Dedupe** by summary + start date: skip events that already exist. Don't update or delete
   existing events unless asked; if an Outlook event's dates changed, flag it in the report
   instead of silently editing.
4. Create missing events:
   - All-day events: `allDay: true`, end date **exclusive** (last day + 1).
   - Timed events: exact times, `timeZone: Europe/Helsinki`, include location.
   - Description: `Synced from Outlook calendar: Runtech RunPro Team` (+ any useful note, e.g.
     "invite not yet answered in Outlook").
5. Verify with a final `list_events` and report created/skipped counts.

The sync is one-way (Outlook → Google) and snapshot-based. Never write back to Outlook.

## Step 5 — visual page (always, unless told "ei sivua")

Render the briefing as a single self-contained HTML file in the **Erika minimalist style**.
This step is part of every full run — deliver the chat report first, then build the page from
the same content:

- Light theme: bg `#ffffff`, soft section bg `#f4f4f2`, text `#1a1a1a`, muted `#8a8a8a`,
  border `#e6e6e2`, accent gold `#c9a04e`, black CTA buttons.
- Font 'Jost' (Google Fonts, system-ui fallback), weights 300–600. Uppercase letterspaced
  headings (h1 `letter-spacing:0.12em`, nav/labels `0.2–0.35em`). Max width 1000px, generous
  whitespace, near-square corners (2px).
- Structure: sticky top nav (anchor links) → hero (date kicker, big headline with one accent-gold
  word + blinking cursor, count line, black CTA) → Punaiset as cards with red pill badges →
  Keltaiset as a 2-col grid with yellow left-border items → Vihreät as a dotted 2-col list →
  Kalenteri as a vertical timeline (gold dots, red dot = ongoing) → gold callout for the week's
  bottleneck → minimal footer.
- Traffic-light colors: red `#c0392b`, yellow `#b8860b`, green `#2e7d52`, each with an ~8%
  alpha soft background for badges.
- Subtle IntersectionObserver fade-ins only; page must be fully readable with JS disabled.
  No external assets beyond the Google Font. Responsive: grids collapse to one column <720px.
- Save to `~/J3s/OUTPUTS/` as
  `YYYY-MM-DD-DailyBriefing-Visual-vNN.html` (bump vNN if same-day file exists), then present
  the file.

## Failure modes

- "Workspace still starting" from bash → wait and retry.
- Outlook window partly off-screen or covered: `open_application` brings it forward; don't
  resize unless necessary.
- A granted-app screenshot hides other apps — that's expected, not an error.
- If "Runtech System" calendar doesn't exist, stop and ask Juhani to create it in Google
  Calendar (the connector cannot create calendars).
- If the mail pass finds nothing red, say so plainly — don't inflate yellows into reds.
