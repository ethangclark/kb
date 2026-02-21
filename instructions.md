# Canon

A goal-driven knowledge base for Claude Code. Uses legal citation patterns and wiki-style organization to provide structured, navigable, citation-backed memory across sessions.

Canon is a directory convention (`.canon/`), a `CLAUDE.md` file, and a set of skills. No compiled binary, no external dependencies. The agent does everything.

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
├── CLAUDE.md                      # Core agent instructions (overview, formats, consistency rules)
├── .claude/
│   └── skills/                    # Skill definitions (one dir per slash command)
│       ├── crystallize/
│       │   └── SKILL.md
│       ├── ingest/
│       │   └── SKILL.md
│       ├── recall/
│       │   └── SKILL.md
│       ├── brief/
│       │   └── SKILL.md
│       ├── check/
│       │   └── SKILL.md
│       ├── triage/
│       │   └── SKILL.md
│       ├── review/
│       │   └── SKILL.md
│       └── lint/
│           └── SKILL.md
└── .canon/
    ├── GOALS.md                   # Goal tree (required)
    ├── plans/                     # Granular sub-goals / action plans
    │   └── {plan}.md
    ├── proposals/                 # Pending changes to GOALS.md or plans (queue, not archive)
    │   └── {proposal}.md
    ├── reference/
    │   ├── INDEX.md               # Navigation index (required)
    │   └── {domain}/
    │       └── {topic}.md
    ├── sources/
    │   ├── src-YYYYMMDD-HHMMSS-slugified-name.md
    │   └── src-YYYYMMDD-HHMMSS-slugified-name.raw.pdf  (optional binary)
    └── unprocessed/              # Source files not yet integrated
        └── ...
```

### Special Files

**`CLAUDE.md`** — Core agent instructions. Contains the directory convention, file formats, citation syntax, addressing rules, and consistency model. This is the shared context that all skills reference. It does *not* contain workflow definitions — those live in skills. Workflow logic is factored into skills so each is self-contained and independently editable. If a workflow misbehaves, you refine that skill's `SKILL.md` — the blast radius is scoped.

**`GOALS.md`** — The goal tree. Hierarchical, abstract. Sub-goals nest until they become indistinguishable from action plans, at which point they live in `plans/`. Can include conflict-resolution steps and decision criteria. Can reference and be referenced by any reference file.

**`proposals/`** — A queue of agent-drafted changes to `GOALS.md` or plan files. Each proposal describes a specific change, cites the source that motivated it, and explains the reasoning. The agent never modifies `GOALS.md` or plan files directly — it writes a proposal instead. `/review` processes the queue: accepted proposals are applied and the proposal file is deleted; rejected proposals are deleted. Nothing in `proposals/` is permanent.

**`reference/INDEX.md`** — Navigation index for reference material. Every reference document must be reachable from here. Entry point for both human browsing and agent traversal during `/recall`. Updated by the agent when reference docs are created, moved, or deleted. Follows the same frontmatter and formatting rules as any other reference file.

---

## Frontmatter and Reference Format

All `.canon/` content files share a common frontmatter shape: `headnote` + `last_updated`, plus any type-specific fields. This keeps parsing uniform and makes every document self-describing for retrieval.

### Source files (`.canon/sources/src-*.md`)

```yaml
---
headnote: "Call with lead investor re: target company. Focus on org changes and updated revenue."
last_updated: 2026-02-21T14:30:00Z
name: investor-call-feb-21
ingested: 2026-02-21T14:30:00Z
---
```

`headnote` summarizes the source's content for retrieval. `last_updated` tracks the most recent modification. `name` is the human label (also used in the filename slug). `ingested` is when the source entered the system.

### Reference files (`reference/**/*.md`)

Reference docs use `§`-numbered sections with inline citations:

```markdown
---
headnote: Acme Services is a distressed regional contractor specializing
  in residential exterior work. Revenue ~$12M trailing. Key risk is
  recent loss of senior project manager and geographic concentration.
last_updated: 2026-02-21T15:44:00Z
---

# Acme Services

## §1 Company Overview

### §1.1 Business Profile
Claim text with citation immediately after.
[Established: src-20260115-091500-acme-cim["anchor start"…"anchor end"]]

### §1.2 Financial Summary
Claim with multiple citations.
[Established: src-20260108-investor-call["from"…"to"];
cf. industries/residential-contracting§2;
but see: src-20260201-investor-update["from"…"to"]]

## §2 Key Personnel

### §2.1 Leadership
Cross-references to other knowledge base sections.
[Established: src-20260115-091500-acme-cim["from"…"to"];
accord: people/deal-lead§2.1;
see generally: frameworks/sde-calculation§3]
```

Rules:

- Subsections nest: `§2` → `§2.1` → `§2.1.1` (depth as needed).
- **Every factual claim must have a citation.** No exceptions.
- The `headnote` is agent-written and kept current. It's the primary signal used during `/recall` traversal.

### Goal and plan files (`.canon/GOALS.md`, `.canon/plans/*.md`)

```yaml
---
headnote: Acquire a residential exterior contractor in the Southeast
  with $8-15M revenue and owner willing to stay 12+ months post-close.
last_updated: 2026-02-21T12:00:00Z
---
```

`headnote` summarizes the goal or plan. `last_updated` tracks the most recent modification.

### Proposal files (`.canon/proposals/*.md`)

```yaml
---
headnote: Split geographic search into Southeast and Mid-Atlantic sub-goals
  based on new market data showing divergent contractor density.
last_updated: 2026-02-21T16:00:00Z
source: src-20260221-143000-investor-call-feb-21
target: GOALS.md
---
```

`headnote` summarizes the proposed change. `source` references the source that triggered it. `target` identifies the file to be modified (`GOALS.md` or a plan file). Proposals are queue items — they are deleted after `/review` processes them (whether accepted or rejected).

### Unprocessed files (`.canon/unprocessed/`)

Files in `unprocessed/` are source material that hasn't been integrated yet. No naming convention or frontmatter is required — they can be in any format. When processed via `/ingest` or `/triage`, the agent creates a proper source file in `sources/` with full frontmatter.

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

The agent maintains consistency during every write operation. After modifying files, it runs `/lint` scoped to the affected files as the final step. The checks are:

1. All citations resolve to existing sources.
2. All `["from"…"to"]` text spans match content in the referenced source.
3. All cross-references (`§` addresses) resolve to existing sections.
4. `INDEX.md` entries match the files that actually exist in `reference/`.
5. Every proposal has a valid `target` that exists.
6. All content files have valid frontmatter for their type.

When a reference doc is updated, everything that cites it must be checked for continued validity. If a downstream reference is invalidated, it must be updated too — recursively — until the knowledge base is settled. The scope of `/lint` expands to cover these downstream files.

---

## Crystallization

Canon requires explicit goal definition before it becomes useful. The `/crystallize` skill drives this process:

1. The user runs `/crystallize` to begin an interactive goal-refinement session.
2. The agent works with the user to build or refine the goal tree in `GOALS.md`, creating sub-goals and plans until the tree is complete and actionable.
3. Important heuristic: Goals are refined hierarchically — sub-goals are clarified until further refinement would be less conducive to root goal achievement than starting execution.
4. Once the goal tree is stable, the system is "crystallized" and ready for ingestion.

---

## Skills

Skills are the entire user interface. Each skill is a `SKILL.md` file in `.claude/skills/<skill-name>/` that defines one workflow. Claude Code discovers them automatically — the skill's `description` frontmatter tells Claude when to invoke it, and users can invoke any skill directly with `/<skill-name>`.

| Skill | What it triggers |
|-------|-----------------|
| `/crystallize` | Interactive goal refinement session |
| `/ingest` | Claim extraction, citation, reference updates, consistency settlement |
| `/recall <query>` | Reasoning-based index traversal to answer a question |
| `/brief [<topic>]` | Context briefing with contradiction surfacing |
| `/check <claim>` | Claim verification against the knowledge base |
| `/triage` | Walk through unprocessed items |
| `/review` | Walk through pending goal/plan proposals |
| `/lint <file> [<file>...]` | Validate consistency of specified files |

### Skill file format

Each skill lives in its own directory under `.claude/skills/` and contains a `SKILL.md` with YAML frontmatter and markdown instructions:

```yaml
---
name: check
description: Verify a specific claim against the Canon knowledge base.
  Use when the user wants to fact-check a statement or confirm whether
  something is supported by ingested sources.
argument-hint: [claim]
---

Verify the claim `$ARGUMENTS` against the knowledge base.

## Steps
1. Locate relevant reference material via INDEX.md traversal.
2. Evaluate whether the claim is supported, contradicted, or unaddressed.
3. Respond with citations for each determination.

## Completion Criteria
- Claim assessed as supported, contradicted, partially supported, or
  not addressed — with citations for each determination.
```

Key frontmatter fields:

| Field | Purpose |
|-------|---------|
| `name` | The `/slash-command` name. If omitted, uses the directory name. |
| `description` | Tells Claude when to invoke the skill automatically. Also shown in autocomplete. |
| `disable-model-invocation` | Set `true` to prevent Claude from auto-invoking (manual `/name` only). |
| `allowed-tools` | Tools Claude can use without per-use approval when the skill is active. |
| `argument-hint` | Hint shown during autocomplete, e.g. `[query]` or `[topic]`. |

See [Claude Code skills documentation](https://code.claude.com/docs/en/skills) for the full reference.

### `/crystallize`

Interactive goal refinement. The agent works with the user to build or refine the goal tree in `GOALS.md`. Creates sub-goals and plans until the tree is complete and actionable enough to begin execution.

**Completion criteria:** `GOALS.md` exists with at least one root goal and enough sub-goal structure that the agent can evaluate incoming information for relevance.

### `/ingest`

Given a source document (provided inline, as a file path, or described by the user) with a name and optional context:

1. Create the source file in `.canon/sources/` with appropriate frontmatter.
2. Read `.canon/GOALS.md` to understand what matters.
3. Extract claims from the source, generating inline citations with text-span anchors.
4. For each claim, determine where it belongs in the reference tree (new doc, existing section, or new section in existing doc).
5. Write or update reference docs with cited claims. Update headnotes.
6. Update `INDEX.md` as needed.
7. Check downstream: read everything that references changed sections. Update if the change invalidates them. Repeat until settled.
8. If the source reveals a more efficient path to a root goal, draft a proposal in `.canon/proposals/` with citations to the triggering source.
9. If relevance to goals is ambiguous, place in `unprocessed/` instead.
10. Run `/lint` on each file created or modified in the above steps.

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

Walk the user through pending items in `.canon/proposals/`. For each proposal, present the proposed change, the target file, the source that triggered it, and the reasoning. The user accepts, modifies, or rejects. Accepted proposals are applied to the target file and the proposal is deleted. Rejected proposals are deleted. The agent settles any downstream effects of accepted changes.

**Completion criteria:** `proposals/` is empty. Any accepted changes applied and KB consistency verified.

### `/lint <file> [<file>...]`

Run the consistency checks from the Consistency Model section against one or more specific files. Example: `/lint reference/deals/acme-services.md proposals/split-geo-search.md`. Linting a file also verifies what it directly references (e.g. checking a reference doc confirms its citations resolve to real source spans).

**Completion criteria:** Full report of errors for the specified files, or confirmation that they are clean.

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

The current design flags contradictions with `But see:` citations and defers resolution to `.canon/GOALS.md` / `.canon/plans/` where it's on the critical path. `/brief` is specced to surface these, but the exact heuristic for "on the critical path" needs definition.

### Recall Depth vs. Cost

The reasoning-based `/recall` traversal is powerful but potentially expensive for deep knowledge bases. Need to think about traversal budgets, caching of recent traversals, or pre-computed "hot paths" based on goal relevance.
