---
name: huomenta
description: "Unified daily morning briefing for Juhani — assembles email action items, today's calendar, crypto pulse, and a paper & board industry scan into one tight, opinionated start-of-day brief. Use this skill whenever Juhani says 'huomenta', 'morning briefing', 'aamupala', 'daily brief', 'start my day', or asks anything that resembles a morning catch-up across his email, calendar, crypto holdings, or paper industry. Trigger even if he only mentions one of the four (e.g. 'just crypto today') — the skill supports partial runs."
---

# Huomenta — Daily Morning Briefing

A unified daily brief covering four pillars: **email action items**, **today's calendar**, **crypto pulse**, and a **paper & board industry scan**. Built around Juhani's actual context — paper technology consultant working internationally for Mondi, Smurfit Westrock, and other major clients; based in Masku/Turku, Finland.

The output is opinionated, tight, and ends with one clear "do this first" recommendation. No filler. No hedge.

---

## Trigger behavior

- **Default:** `huomenta` (or equivalent) runs the full briefing — all four pillars.
- **Partial:** Phrases like "huomenta crypto", "just my calendar", "email action items only" run only that pillar. Skip the rest.
- Always include today's date in the header.

---

## Pillar 1 — Email (action items only)

Goal: surface what needs Juhani's action today. Not a recap, not an inbox tour. Action triage.

### Gmail (personal)

1. Read all **unread** messages in the label/folder named **`1. Action Items`**.
2. For each unread message, capture: sender, one-line subject summary, what the sender is asking for, and a suggested action.
3. Prioritize by:
   - **🔴 Today** — time-sensitive, deadline today or already overdue.
   - **🟡 This week** — needs response in the next few days.
   - **🟢 Whenever** — useful but not urgent.
4. If there are no unread messages, say so in one line. Don't pad.

### Outlook (work) — desktop application only

**Critical constraint:** Do NOT use the Microsoft 365 MCP connector for work email. Juhani's work setup requires the desktop Outlook application. If the runtime environment cannot drive the desktop app or read its content, say so plainly and ask Juhani to paste the unread items — do not silently fall back to the M365 MCP.

1. Read all **unread** messages in:
   - The **`Action Items`** folder
   - The **`Saapuneet`** (Inbox) folder
2. Apply the same triage as Gmail: sender, ask, suggested action, priority.
3. Flag anything related to active client projects (Mondi, Smurfit Westrock, RunPro turbo compressor work, etc.) — these jump to the top regardless of urgency.

### Email output format

```
## 📧 Email — Action required

### Gmail (personal) — [N] unread in Action Items
🔴 [Sender]: [one-line ask] → [suggested action]
🟡 [Sender]: [one-line ask] → [suggested action]

### Outlook (work) — [N] unread in Action Items + Saapuneet
🔴 [Sender]: [one-line ask] → [suggested action]
🟡 [Sender]: [one-line ask] → [suggested action]
```

If a section has no unread items: `_Inbox zero in [folder]._`

---

## Pillar 2 — Calendar (today only)

Goal: walk through today's schedule with light commentary. Not a copy-paste of titles — useful framing.

### Google Calendar (personal)

1. Pull all events on **today's date**.
2. For each: time, title, location/link if relevant, one-line note (e.g. *"Pack kayaking gear night before"* or *"Balu vet — bring previous prescription"*).

### Outlook (work) — desktop application only

Same constraint as work email: desktop application only, not the M365 MCP. If unavailable, ask Juhani to share his calendar view.

1. Pull today's events from these calendars:
   - **`Omat kalenterit`** (My calendars)
   - **`Calendar`** (default/primary)
   - **`Runtech RunPro Team`** (shared team calendar)
2. List in chronological order, deduplicating where the same event appears in multiple calendars.
3. Note the calendar source for each event (e.g. *"[RunPro Team]"*) so Juhani knows where it came from.
4. Flag conflicts — overlapping events get a `⚠️`.

### Calendar output format

```
## 📅 Today — [Day], [Date]

### Personal (Google)
- 07:30  Walk Balu (45 min) — _light rain expected, take long jacket_
- 18:00  Kayaking session, Naantali — _gear packed?_

### Work (Outlook desktop)
- 09:00  Mondi Świecie pre-call _[Calendar]_ — _agenda: Q2 audit scope_
- 11:00  RunPro service review _[RunPro Team]_
- ⚠️ 14:00–15:00 overlap: Smurfit Piteå call _[Calendar]_ vs. internal IRCO review _[Omat kalenterit]_
```

---

## Pillar 3 — Crypto pulse

Goal: 2–4 things worth knowing from Juhani's curated sources. Not a market summary, not price commentary unless something has genuinely moved.

### Source weighting (Juhani's stated preferences)

- **CoinDesk** + **The Block** — primary sources. Treat as authoritative for news and on-chain developments.
- **Decrypt** — readable context, good for "what does this actually mean" framing.
- **Blockworks** — market thinking and macro-flavored analysis. Use for *why now* angles.
- **Cointelegraph** — fast radar, **not** the final truth machine. If a story only appears on Cointelegraph and not on CoinDesk or The Block, mark it as `[unconfirmed]` and note the single source.

### Method

1. Run web searches against these sources for the last 24–48 hours.
2. Filter for: regulatory/policy moves, ETF/institutional flows, major protocol or exchange events, anything material to **BTC** or **SOL** specifically (per Juhani's portfolio split).
3. Skip: routine price commentary, "altcoin gem" content, influencer noise.
4. Discard items that don't pass the source weighting test.

### Crypto output format

```
## 🪙 Crypto pulse

- **[Headline]** _([source])_ — [1-2 sentence what + why it matters].
- **[Headline]** _([source])_ — [1-2 sentences].
- **[Headline]** _(Cointelegraph, unconfirmed)_ — [1 sentence + caveat].
```

If nothing material: `_Quiet 24h. BTC and SOL within normal range._` Don't manufacture findings.

---

## Pillar 4 — Industry scan (paper & board)

Goal: surface what matters today in Juhani's professional world — paper machine technology, runnability, energy efficiency, and the major clients/players in his orbit.

### Search areas (run 4–5 web searches)

1. **Client news** — Mondi, Smurfit Westrock specifically. Earnings, plant closures, capex, leadership changes.
2. **Major peers** — UPM, Stora Enso, Metsä, International Paper, Sappi, Holmen, Essity, Norske Skog. Anything market-moving.
3. **Technology and runnability** — paper machine rebuilds, vacuum systems, energy efficiency, dewatering, web stabilization.
4. **Market signals** — containerboard pricing, capacity moves, energy/raw material cost trends, trade and tariff developments affecting EU/NA pulp & paper.
5. **Finland & Nordics** — UPM, Stora Enso, Metsä, Valmet, Andritz news; Finnish industry-specific items.

### Scoring

Rate each finding on three dimensions:

| Criterion | Scale | What it measures |
|---|---|---|
| Relevance to Juhani's work | 1–5 | How directly does this touch his clients, technology stack, or commercial leverage? |
| Buzz | 1–5 | How much industry attention is this getting right now? |
| Timeliness | 1–3 | 3 = today only; 2 = this week; 1 = evergreen |

**Max score: 13.** Discard anything below 7. Surface only the top items.

### Urgency tags

- 🔴 **Act today** — time-sensitive (e.g. earnings call live, deadline, named client event).
- 🟡 **Act this week** — relevant context for client touchpoints in the next few days.
- 🟢 **Evergreen** — useful background, queue it.

### Industry scan output format

```
## 🏭 Industry scan

### 1. [Headline]
**Score:** XX/13 | **Urgency:** [emoji]
**What happened:** [2-3 sentence summary]
**Why it matters for your work:** [1 sentence — connect to clients / technology / commercial leverage]
**Conversation starter:** "[A ready-to-use opening line for client touchpoint or memo]"
**Use angle:** [How to use this — memo, client outreach, project framing, internal note]

### 2. ...
### 3. ...

### Quick Hits
- [Brief item 1 — what + why notable]
- [Brief item 2]
- [Brief item 3]
```

---

## Briefing assembly

When all pillars are run, assemble in this order:

```
# Huomenta — [Day], [Date]

## ⚡ What needs you today
[3 bullets max — cherry-pick the most urgent items across all four pillars.
This is the "if you do nothing else, do these" bar. Be ruthless.]

---

## 📧 Email — Action required
[Pillar 1 output]

## 📅 Today — [Date]
[Pillar 2 output]

## 🪙 Crypto pulse
[Pillar 3 output]

## 🏭 Industry scan
[Pillar 4 output]

---

## My take
[1-2 sentences. Pick ONE thing across all four pillars that should be done first today. No hedge. No "you might consider." Direct stance.]

---

Want me to: draft a reply to any of the action emails, dig deeper on a specific industry item, or draft a memo / client outreach note?
```

---

## Rules

- Always include today's date in the header.
- Be opinionated. Tell Juhani what to do, don't lay out neutral options.
- Verify facts via web search. If a search result is ambiguous, note the uncertainty — don't guess.
- Do not pad. If a pillar has nothing material, say so in one line and move on.
- Hard length limits per pillar:
  - Email: max 8 items total across personal + work
  - Calendar: as many events as exist, but one line each
  - Crypto: max 4 items
  - Industry: max 3 top items + max 5 quick hits
- The "What needs you today" bar at the top is the most important section. If Juhani only reads three lines, those three lines should be enough.
- For Outlook work email and calendar: respect the desktop-app-only constraint. Do NOT use the Microsoft 365 MCP. If the runtime can't access the desktop app, say so plainly and ask Juhani to paste content — never silently fall back.
- Cointelegraph items must be marked `[unconfirmed]` if not corroborated by CoinDesk or The Block.
- Files Juhani might want saved (the briefing as a Word doc, a memo draft, etc.) go to `/Users/j3s/Documents/Claude/OUTPUTS/`.
