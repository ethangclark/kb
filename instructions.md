# Canon

A goal-driven knowledge base for Claude Code. Uses legal citation patterns and wiki-style organization to provide structured, navigable, citation-backed memory across sessions.

Canon is a directory convention (`.canon/`), a `CLAUDE.md` file, and a set of slash commands. No compiled binary, no external dependencies. The agent does everything.

---

## The Problem

LLMs have no durable memory. The workarounds all have fundamental limitations:

- **Vector/RAG search** matches on *similarity*, not *relevance*. A sentence about "bridge financing" might retrieve content about literal bridges.
- **MEMORY.md files** hit context limits and carry irrelevant baggage.
- **Manual prompt construction** doesn't scale and depends on the user knowing what's relevant.

Meanwhile, two well-established systems for building navigable, citation-backed knowledge bases already exist: **wikis** (hierarchical, interlinked reference documents) and **legal citation** (provenance, contradiction tracking, cross-referencing). Canon combines both.

---

## Design Principles

1. **Goals first.** "Perfect memory" is unbounded. Structure only makes sense relative to what you're trying to accomplish. Canon requires explicit goal definition so it can optimize what to retain, how to organize it, and what to surface.
2. **Markdown-native.** All state lives in `.md` files. Humans can read them. Git can version them. Any tool can consume them.
3. **Every claim is cited.** No uncited assertions in reference material. If something is stated, you can trace it back to a source.
4. **Zero infrastructure.** No CLI binary, no server, no database. Just a directory convention and a `CLAUDE.md` that teaches Claude Code how to operate on it. A working knowledge base is one `git clone` away.

---

## Directory Structure

```
project-root/
├── CLAUDE.md                      # Slash command definitions and workflow instructions
└── .canon/
    ├── goals/
    │   ├── GOALS.md               # Goal tree (required)
    │   ├── plans/                 # Granular sub-goals / action plans
    │   │   └── {plan}.md
    │   └── proposals/             # Agent-drafted goal/plan revisions awaiting user review
    │       └── {proposal}.md
    ├── reference/
    │   ├── INDEX.md               # Root navigation index (required)
    │   └── {domain}/
    │       ├── INDEX.md           # Local navigation (if needed)
    │       └── {topic}.md
    ├── sources/
    │   ├── src-YYYYMMDD-HHMMSS-slugified-name.md
    │   └── src-YYYYMMDD-HHMMSS-slugified-name.raw.pdf  (optional binary)
    └── unprocessed/
        ├── src-YYYYMMDD-HHMMSS-slugified-name.md
        └── src-YYYYMMDD-HHMMSS-slugified-name.raw.pdf  (optional binary)
```

### Special Files

**`CLAUDE.md`** — The control plane. One section per slash command, defining workflow steps, constraints, and completion criteria. This is the entire "application logic" of Canon.

**`goals/GOALS.md`** — The goal tree. Hierarchical, abstract. Sub-goals nest until they become indistinguishable from action plans, at which point they live in `goals/plans/`. Can include conflict-resolution steps and decision criteria. Can reference and be referenced by any reference file.

**`goals/proposals/`** — Agent-drafted revisions to the goal tree or plans. When ingestion reveals a more efficient path to a root goal, the agent writes a proposal here with citations to the source that triggered it. The user reviews via `/review`. The agent never autonomously modifies `GOALS.md` or plan files.

**`reference/INDEX.md`** — Root-level navigation index. Every reference document must be reachable from here, either directly or through nested `INDEX.md` files in subdirectories. Entry point for both human browsing and agent traversal during `/recall`. Updated by the agent when reference docs are created, moved, or deleted.

**Nested `INDEX.md` files** — Subdirectories may contain their own `INDEX.md` for local navigation. The root `INDEX.md` links to these. The full index tree must be consistent: no orphaned docs, no dead links.

---

## Frontmatter

### Source files (`.canon/sources/src-*.md` and `.canon/unprocessed/src-*.md`)

```yaml
---
name: investor-call-feb-21
ingested: 2026-02-21T14:30:00Z
context: "Call with lead investor re: target company. Focus on org changes and updated revenue."
---
```

`name` is the human label (also used in the filename slug). `ingested` is when it entered the system. `context` is optional user-provided guidance that informs how the agent extracts claims and prioritizes content during ingestion. Unprocessed files use the same schema — they are source files that haven't been integrated yet. When processed, they move to `sources/` with frontmatter intact.

### Reference files (`reference/**/*.md`, excluding `INDEX.md`)

```yaml
---
headnote: Acme Services is a distressed regional contractor specializing
  in residential exterior work. Revenue ~$12M trailing. Key risk is
  recent loss of senior project manager and geographic concentration.
last_updated: 2026-02-21T15:44:00Z
---
```

`headnote` is an agent-written summary optimized for retrieval — the primary signal used during `/recall` traversal. `last_updated` is when the doc was last substantively changed.

### Goal and plan files (`goals/**/*.md`, excluding `proposals/`)

```yaml
---
last_updated: 2026-02-21T12:00:00Z
---
```

Just the timestamp. The goal tree's structure is the content — no headnote needed since `GOALS.md` is always read in full and plans are navigated from it directly.

### INDEX.md files

No frontmatter. Purely structural/navigational, auto-maintained by the agent.

---

## Reference Document Format

YAML frontmatter → `§`-numbered sections → prose with inline citations.

```markdown
---
headnote: 2-4 sentence summary optimized for LLM retrieval.
  Key conclusions and current state.
last_updated: 2026-02-15T13:44:22Z
---

# Document Title

## §1 Top-Level Section

### §1.1 Subsection
Claim text with citation immediately after.
[Established: src-20260115-091500-acme-cim["anchor start"…"anchor end"]]

### §1.2 Another Subsection
Claim with multiple citations.
[Established: src-20260108-investor-call["from"…"to"];
cf. industries/residential-contracting§2;
but see: src-20260201-investor-update["from"…"to"]]

## §2 Next Section

### §2.1 Subsection
Cross-references to other knowledge base sections.
[Established: src-20260115-091500-acme-cim["from"…"to"];
accord: people/deal-lead§2.1;
see generally: frameworks/sde-calculation§3]
```

### Rules

- Subsections nest: `§2` → `§2.1` → `§2.1.1` (depth as needed).
- **Every factual claim must have a citation.** No exceptions.
- The `headnote` is agent-written and kept current. It's the primary signal used during `/recall` traversal.

### Addressing

| Type | Format | Example |
|------|--------|---------|
| Section | `knowledge-path§section` | `deals/acme-services§2.1` |
| Whole source | `src-YYYYMMDD-HHMMSS-slug` | `src-20260115-091500-acme-cim` |
| Text span | `source["from"…"to"]` | `src-20260115-091500-acme-cim["Revenue was"…"per year"]` |
| Historical | `source@commit["from"…"to"]` | `src-20260115-091500-acme-cim@a3f7c2["from"…"to"]` |

### Citation Signals

| Signal | Meaning |
|--------|---------|
| `Established:` | Primary source for this claim |
| `Updated:` | Newer information modifying the original |
| `Supersedes:` | This replaces a prior claim entirely |
| `Cf.` | Compare with (related but different angle) |
| `But see:` | Contradictory or qualifying information |
| `Accord:` | Consistent with / corroborated by |
| `See generally:` | Background or framework reference |

---

## Consistency Model

The agent maintains consistency during every write operation. This means:

1. When a reference doc is updated, everything that cites it must be checked for continued validity.
2. If a downstream reference is invalidated, it must be updated too — recursively — until the knowledge base is settled.
3. All `INDEX.md` files must reflect the current directory structure.
4. No orphaned documents (unreachable from root `INDEX.md`).
5. No dead links (references to nonexistent sections or sources).

The agent enforces this by running a validation pass as the final step of every write workflow (`/ingest`, `/triage` → process, `/review` → accept). The validation reads all `INDEX.md` files, walks citations and cross-references, and confirms everything resolves. If anything is broken, the agent fixes it before declaring the operation complete.

If the knowledge base grows large enough that agent-based validation becomes unreliable or slow, that's the signal to extract a dedicated lint tool. Until then, the agent handles it.

---

## Crystallization

Canon requires explicit goal definition before it becomes useful. The `/crystallize` slash command drives this process:

1. The user runs `/crystallize` to begin an interactive goal-refinement session.
2. The agent works with the user to build or refine the goal tree in `GOALS.md`, creating sub-goals and plans until the tree is complete and actionable.
3. Goals are refined hierarchically — sub-goals are clarified until further refinement would be less productive than starting execution.
4. Once the goal tree is stable, the system is "crystallized" and ready for ingestion.

**Bootstrap shortcut:** If `/ingest` is run before crystallization, the agent either infers goal relevance from the document content (drafting a `GOALS.md` proposal for user review) or places the document in `unprocessed/` if the connection to goals is ambiguous.

---

## Slash Commands

Slash commands are the entire user interface. Each maps to a section in `CLAUDE.md` that defines the workflow.

| Command | What it triggers |
|---------|-----------------|
| `/crystallize` | Interactive goal refinement session |
| `/ingest` | Claim extraction, citation, reference updates, consistency settlement |
| `/recall <query>` | Reasoning-based index traversal to answer a question |
| `/brief [<topic>]` | Context briefing with contradiction surfacing |
| `/check <claim>` | Claim verification against the knowledge base |
| `/triage` | Walk through unprocessed items |
| `/review` | Walk through pending goal/plan proposals |
| `/lint` | Validate KB consistency (citations, links, indexes, orphans) |

### `/crystallize`

Interactive goal refinement. The agent works with the user to build or refine the goal tree in `GOALS.md`. Creates sub-goals and plans until the tree is complete and actionable enough to begin execution.

**Completion criteria:** `GOALS.md` exists with at least one root goal and enough sub-goal structure that the agent can evaluate incoming information for relevance.

### `/ingest`

Given a source document (provided inline, as a file path, or described by the user) with a name and optional context:

1. Create the source file in `.canon/sources/` with appropriate frontmatter.
2. Read `goals/GOALS.md` to understand what matters.
3. Extract claims from the source, generating inline citations with text-span anchors.
4. For each claim, determine where it belongs in the reference tree (new doc, existing section, or new section in existing doc).
5. Write or update reference docs with cited claims. Update headnotes.
6. Update `INDEX.md` files as needed.
7. Check downstream: read everything that references changed sections. Update if the change invalidates them. Repeat until settled.
8. If the source reveals a more efficient path to a root goal, draft a proposal in `goals/proposals/` with citations to the triggering source.
9. If relevance to goals is ambiguous, place in `unprocessed/` instead.
10. Run `/lint` as a final check.

**Completion criteria:** Source file in `sources/`. All extracted claims cited in reference docs. `INDEX.md` current. No broken links or orphaned docs. Any goal-revision proposals drafted.

### `/recall <query>`

1. Start at `reference/INDEX.md`.
2. Read the index and headnotes to decide which branches are relevant to the query.
3. Traverse into relevant documents, reading `§`-sections as needed.
4. If a section cross-references another document, follow the link if judged relevant.
5. Continue until there's enough context to answer, or relevant paths are exhausted.
6. Respond with citations back to the reference material consulted.

This is more expensive than vector search but dramatically more accurate — the agent makes *relevance* judgments, not similarity matches, guided by the goal-aware structure of the index.

**Completion criteria:** Query answered with specific citations to reference material, or explicit statement that the KB doesn't contain relevant information.

### `/brief [<topic>]`

Generate a context briefing for a topic or the full KB. Similar traversal to `/recall`, but optimized for "bring me up to speed" rather than answering a specific question. Proactively surfaces unresolved contradictions (`But see:` citations) that are on the critical path of active goals.

**Completion criteria:** Briefing delivered with citations. Contradictions flagged if any exist on the critical path.

### `/check <claim>`

Verify a specific claim against the KB. The agent locates relevant reference material, evaluates whether the claim is supported, contradicted, or unaddressed, and responds with citations.

**Completion criteria:** Claim assessed as supported, contradicted, partially supported, or not addressed — with citations for each determination.

### `/triage`

Surface items in `unprocessed/` ordered by age (oldest first) and walk the user through each:

- **Process now** — run the `/ingest` workflow, integrating into reference material.
- **Defer** — leave it in `unprocessed/`.
- **Discard** — remove from `unprocessed/` (source file retained in `sources/`).

**Completion criteria:** All items in `unprocessed/` addressed or user ends the session.

### `/review`

Walk the user through pending items in `goals/proposals/`. For each proposal, present the suggested change, the source that triggered it, and the reasoning. The user accepts, modifies, or discards. Accepted proposals are applied to `GOALS.md` or plan files, and the agent settles any downstream effects.

**Completion criteria:** All proposals addressed. Any accepted changes applied and KB consistency verified.

### `/lint`

Validate knowledge base consistency. The agent reads the full `.canon/` structure and checks:

- All citations resolve to existing sources.
- All `["from"…"to"]` text spans match content in the referenced source.
- All cross-references (`§` addresses) resolve to existing sections.
- All `INDEX.md` files are complete (no orphaned docs, no dead links).
- All reference docs have valid frontmatter (headnote + last_updated).
- All source files have valid frontmatter (name + ingested).

Reports errors if any. This is the same validation the agent runs at the end of every write operation, exposed as a standalone command for manual use or verification.

**Completion criteria:** Full report of errors, or confirmation that the KB is clean.

---

## CLAUDE.md Structure

`CLAUDE.md` is the entire application. It's organized with one section per slash command plus a preamble that covers the directory convention, file formats, citation syntax, and consistency rules.

```markdown
# Canon — Knowledge Base Agent Instructions

## Overview
Canon is a goal-driven knowledge base stored in `.canon/`.
[Directory structure, frontmatter specs, reference doc format,
addressing syntax, citation signals — all from this PRD.]

## Consistency Rules
[The consistency model: what the agent must verify after every write.]

## /crystallize
### Purpose
Build or refine the goal tree through interactive conversation.

### Steps
1. ...

### Constraints
- ...

### Completion Criteria
- GOALS.md exists with at least one root goal.
- Sub-goal structure is sufficient to evaluate incoming information.

## /ingest
### Purpose
Store a source document and integrate its claims into the knowledge base.

### Steps
1. Create source file in `.canon/sources/` ...
...

### Constraints
- Never modify GOALS.md or plan files directly. Draft proposals instead.
- Every factual claim must have an inline citation.
- Run /lint as the final step.

### Completion Criteria
- Source file exists in `sources/`.
- All extracted claims are cited in reference docs.
- INDEX.md files reflect any new or moved documents.
- /lint passes with no errors.

## /recall
...

## /brief
...

## /check
...

## /triage
...

## /review
...

## /lint
...
```

Each section is self-contained. If a workflow misbehaves, you refine that section — the blast radius is scoped. The `CLAUDE.md` is itself version-controlled, so workflow changes are tracked alongside the knowledge they operate on.

---

## Unanswered Questions

### Knowledge Decay / Staleness

How does the system know when reference material is stale? If a CIM was ingested six months ago and the underlying financials have changed, reference sections built from it might be silently outdated.

Possible approaches:

- **Temporal metadata on claims.** Each cited claim could carry an `as_of:` date derived from the source. `/lint` could flag claims older than a configurable threshold.
- **Source expiry hints.** The ingestion workflow could tag sources with an expected validity window (e.g., "financial statements: ~12 months", "org chart: ~6 months"). Reference docs built from expiring sources would inherit the shortest window.
- **Decay as a goal-tree concern.** If a goal depends on time-sensitive information, the goal spec itself could encode a refresh cadence, and `/triage` could surface items due for refresh.

### Ingestion Cost at Scale

Every `/ingest` requires multi-step agent reasoning: claim extraction, citation generation, reference doc updates, consistency propagation, and potentially goal revision proposals. For heavy use with frequent documents, this could be slow and token-intensive. Need to evaluate whether batching or summarization tiers make sense.

### Conflict Resolution Discovery

The current design flags contradictions with `But see:` citations and defers resolution to `goals/GOALS.md` / `goals/plans/` where it's on the critical path. `/brief` is specced to surface these, but the exact heuristic for "on the critical path" needs definition.

### Recall Depth vs. Cost

The reasoning-based `/recall` traversal is powerful but potentially expensive for deep knowledge bases. Need to think about traversal budgets, caching of recent traversals, or pre-computed "hot paths" based on goal relevance.

### When to Extract a Lint Tool

Agent-based `/lint` works fine for small knowledge bases. As the KB grows, deterministic validation may become worth extracting into a standalone tool for speed and reliability. The trigger is when `/lint` starts taking noticeably long or producing inconsistent results across runs.
