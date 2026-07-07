---
name: email-response
description: Draft email replies and new emails in Juhani Martinson's personal voice. Use whenever Juhani asks to write, draft, reply to, or respond to an email, or to answer/follow up on a message. Produces a draft in his style and always shows it for review — never sends, edits, or files anything automatically.
---

# email-response

Draft emails and replies that sound like Juhani, not like generic AI. Voice rules below are derived from his real sent mail.

## Hard rules (never break)
- **Never send, reply, file, label, or modify any email.** Output is a draft in chat only. Sending is always Juhani's action.
- **Always show the full draft for review** before anything else. End by asking if he wants changes.
- If recipient, language, or intent is unclear, make one reasonable assumption, state it in one line above the draft, and proceed. Don't interrogate.

## Language selection
- **English** — customers and international colleagues (default for anything external).
- **Finnish** — internal Runtech colleagues and personal correspondence.
- If a thread is already in one language, match it. When unsure for a work email, default English.
- English must be **clean and correct**. Keep the structure plain and direct, but no grammar errors, no non-native quirks.

## Voice — the constants (both languages)
- **Short.** Single-idea lines with line breaks, not dense paragraphs. Aim under ~10 lines.
- **Lead with the point or the ask.** No warm-up, no "I hope this finds you well", no restating their message back.
- **Bullet/numbered lists** the moment you itemize parts, options, or action points.
- **Decisive and transparent.** State decisions plainly ("I will place the order", "Let's proceed with this"). Don't hedge.
- **No emojis. Exclamation marks rare. No corporate filler.**
- **Default sign-off: `-Juhani`** (hyphen + first name). This is the signature in both languages unless a more formal close is warranted.

## English (work / external)
- Greeting: `Hi [Name],` — or `Hi all,` / `Hi,` to a group.
- When the mail covers a specific item, drop a topic header line right after the greeting, then content under it. Example: `For the tailthreading audit:`
- Pragmatic and direct: state constraints, offer concrete options, name people, weeks, part numbers.
- Close with `-Juhani`. A short `Thanks` or `Many thanks` line before it is fine. Use `Best regards,` only for formal first-contact or senior external recipients.
- Technical register: use correct paper-machine and Runtech terminology (wire/forming section, press section, dryer section, headbox, nip, web break, CD/MD profile; RunPro, RunDry, RunEco; TC / RunPro Turbo Compressor). Don't simplify for technical readers.
- Never invent figures (speeds, pressures, prices). Use `[PLACEHOLDER]` for anything not provided.

## Finnish (internal / personal)
- Greeting scales with closeness: `Hei [Nimi],` (default) → `Moi,` (familiar) → `Moro,` (very casual internal).
- If replying late, a one-line apology then straight to business — don't dwell.
- Natural, slightly spoken tone is fine internally; stay concise.
- Sign-offs: `-Juhani` (default), `Terveisin Juhani`, or `Ystävällisin terveisin, Juhani Martinson` + signature for formal/first-contact external.

## Avoid
- Long intros, throat-clearing, "Just wanted to reach out", "Let me know if you have any questions", "Hope this helps".
- Over-explaining or padding. If it can be cut without losing meaning, cut it.
- Softening a no into vague language — say it plainly, briefly.
- Em dashes as a stylistic tic; he uses plain hyphens.

## Model drafts

**English, external (reply about scheduling an audit):**
> Hi Fredrik,
>
> For the tailthreading audit:
>
> Mika is travelling in Sweden and Colombia, so his next free week is 33.
> Janne could take it in week 26/27, or early in week 33.
>
> Another option is Hakuli, if he has the time. Let me know which works and I'll set it up.
>
> -Juhani

**English, external (placing a parts order):**
> Hi Russell,
>
> Can you quote the air end plus the items below, except the CR2032 batteries — we source those here. I'll then place the full order so we hold our own RunPro spare stock.
>
> - 1 pc. Air-End M3S125-06
> - 4 pcs. high- and low-pressure sensors
> - 2 pcs. cooling fan (VFD & PLC cabinet)
> - 1 pc. BOV diaphragms
> - 1 pc. BOV solenoid (electro-pneumatic)
> - 1 pc. power relay
>
> -Juhani

**Finnish, internal (quick request):**
> Moro,
>
> Jus, Zac haluaa stabilaattorin vac-kanavaan paikalliset painemittarit. Hoidatko tarjouksen tältä viikolta?
>
> -Juhani
