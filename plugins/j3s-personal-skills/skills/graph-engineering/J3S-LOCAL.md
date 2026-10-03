# Local addendum — graph engineering in J3s's system

Read this before applying the skill to anything of J3s's. The upstream skill is generic; this
file states what is already implemented, what is deliberately *not* being done, and the one
hard confidentiality boundary.

Vendored 2026-08-03 from https://github.com/codejunkie99/graph-engineering (MIT, @Av1dlive).
Upstream is a single-commit distillation — a well-organised reading list and prompt pack, not a
framework. Nothing here executes code. Treat its claims as guidance, not as a specification.

---

## 1 · The task-graph half is already implemented — do not re-derive it

The Balu harness (`~/J3s/`) was audited against `references/task-graphs.md` on
2026-08-03 and brought into compliance. **`HARNESS.md` §7a is the authoritative local version.**
Summary of where each rule lives:

| Upstream rule | Local implementation |
|---|---|
| No fake edges | `ROUTING.md` Step 2 — test the edge before sequencing; fan out in one tool-call block |
| The diamond | `ROUTING.md` Step 4 — **Tarkka** (*is it correct?*) ∥ **Lähde** (*is it traceable?*), fresh contexts, Balu owns the merge, either FAIL blocks |
| The stop rule | `HARNESS.md` §7a rule 3 — fan out gathering and verification; **never** fan out the authoring of a report, memo, FAT/SAT, functional description, quote or customer email |
| The human gate | Global `CLAUDE.md` "Ask first" — irreversible edges only |
| Loop cap | `HARNESS.md` §4d — `max_runs` 3 + `PROGRESS.md` dead-loop hash |
| One writer per file | `HARNESS.md` §7 — parallel branches write `WORK/<Task>/branch-<n>.md`; only the merge owner writes the shared artifact |
| Routing in written steps | `ROUTING.md` |
| Agent spawn cap | `HARNESS.md` §4b — delegation depth 1 |

**Do not restructure the harness around this skill.** If the skill and `HARNESS.md` disagree,
`HARNESS.md` wins — it is grounded in measured local behaviour (see `SYSTEM/team/LESSONS.md`).

## 2 · The knowledge-graph half is a pilot, not a commitment

J3s has no knowledge graph today. What exists instead:

- `PROJECTS/Knowledgebase/` — compiled wiki markdown + BM25/LSA hybrid search. **Document
  retrieval**, good at single-hop ("what do we know about X").
- `graphify-out/` — an AST/code graph over the file tree. Not domain entities.
- `MEMORY.md` files — flat facts, root and per-project.

The gap is multi-hop: *"which mills have a TC of this vintage, which of my five were on site,
what failed, and what did we promise."* That is re-derived by hand today from `SYSTEM/state/email.db`,
`SYSTEM/state/wrike.db`, the KB and mill reports.

Pilot lives at `WORK/KG-Pilot/`. **Honour the upstream honesty test** (`/kg-rag`): if the
graph does not beat `kb_search.py` on the pre-written multi-hop eval set, drop it. A graph that
is not clearly winning is a maintenance liability, not an asset.

Two stages are where this dies, per the course — do not let them be skipped:
- **Stage 3 (ontology)** — types and relations before any extraction.
- **Stage 8 (fusion)** — the local problem is already visible: one customer mill appears under an
  abbreviated name in filenames, differently in Wrike, differently again in email and in the
  KB. Entity resolution across those four is the real work.

## 3 · Confidentiality — the boundary that overrides everything here

- **Stays local.** The KB search engine is offline; the pilot stays offline too. No mill data,
  customer material, or Runtech document goes to a new service without J3s clearing that
  specific job — same rule as the Codex contractor in `HARNESS.md` §4c.
- **Never invent a fact to fill a node.** The skill's provenance rule and J3s's `[PLACEHOLDER]`
  rule are the same rule. Every node and edge carries `source` + `extracted_at` + confidence, or
  it does not enter the graph.
- **⛔ OEM confidentiality applies to graph output too.** The RunPro Turbo Compressor's OEM
  must never be named in customer-facing output (the name is kept out of this public repo too). A knowledge graph makes this *worse*, not
  better: GraphRAG serialisation emits `(head)-[REL {source}]->(tail)` lines with provenance
  attached, which is exactly the citation path that leaks the name (`SYSTEM/team/LESSONS.md` L-001).
  **Any KG serving layer must run the OEM-name gate (in `SYSTEM/engine/gates/`) on its output before that output reaches
  a customer-audience document.** Internal use may name it freely — the boundary is audience,
  not folder.

## 4 · What to use this skill for, day to day

- Designing or reviewing a task graph → `references/task-graphs.md`, then check against
  `HARNESS.md` §7a.
- Building or critiquing an ontology → `references/modeling.md`, the `/kg-scope` and
  `/kg-schema` blocks in the upstream `WORKFLOWS.md`.
- Deduplicating customer / mill / machine names → `references/fusion-and-llm.md`.
- Learning the discipline → teaching mode, using the KG pilot as the running example.

Not for: replacing `kb_search.py` for single-hop lookups, or turning every workflow into a graph.
The upstream text is explicit that most one-off prompts and single-document answers do not need
one.
