# j3s-skills — personal Claude marketplace

One marketplace, added once, that makes the same skills appear in **both** Claude Cowork
and Claude Code.

## Plugins in this marketplace

- **mattpocock-skills** — external reference to `mattpocock/skills` (tracks `main`).
- **j3s-personal-skills** — vendored in this repo: `outlook-briefing`,
  `outlook-mail-triage`, `outlook-calendar-sync`, `email-response`, `graph-engineering`.
  Edit these under `plugins/j3s-personal-skills/skills/`.
  - `graph-engineering` is **third-party, vendored unmodified** from
    [codejunkie99/graph-engineering](https://github.com/codejunkie99/graph-engineering) (MIT,
    2026-08-03) — its `LICENSE` travels with it. The one local addition is `J3S-LOCAL.md`,
    which maps the upstream rules onto the Balu harness and states the confidentiality
    boundary for any knowledge graph built over Runtech material. Read that file first.

## Why this exists

`mattpocock/skills` is a *plugin*, not a *marketplace*. Claude Code can pick up its skills
as loose files (via the `skills.sh` installer), but Cowork only installs from a marketplace.
This repo is that marketplace — it points at Matt's plugin, and also carries your own
skills, so both surfaces install the identical thing from the identical source.

## One-time setup

1. Create a **new public GitHub repo** (e.g. `j3s-skills-marketplace`) and push these files.
   The folder layout must stay exactly:

   ```
   .claude-plugin/marketplace.json
   README.md
   ```

2. **Claude Code** (interactive session):

   ```
   /plugin marketplace add j3s/j3s-skills-marketplace
   /plugin install mattpocock-skills@j3s-skills
   /plugin install j3s-personal-skills@j3s-skills
   ```

   (Replace `j3s` with your GitHub username. First remove the old loose install of Matt's
   skills if you want a single source of truth — otherwise the skill exists twice.)

3. **Cowork:** Customize (left sidebar) → Plugins → add marketplace by URL →
   `https://github.com/j3s/j3s-skills-marketplace` → install **mattpocock-skills** and
   **j3s-personal-skills**.

4. Run `/setup-matt-pocock-skills` on either side to configure Matt's skills per repo.

## Editing / adding your own skills

`j3s-personal-skills` is vendored in this repo — edit the `SKILL.md` files under
`plugins/j3s-personal-skills/skills/` and push; bump `version` in its `plugin.json` so
Claude picks up the update. To add a new personal skill, create a new folder with a
`SKILL.md` there and add its path to the `skills` array in that `plugin.json`.

## Pinning a version (optional)

`ref: "main"` tracks the latest. To lock a specific release instead, replace the `source`
block with a pinned commit:

```json
"source": {
  "source": "url",
  "url": "https://github.com/mattpocock/skills.git",
  "sha": "<commit-sha-from-a-release-tag>"
}
```
