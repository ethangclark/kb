# Knowledge base (kb)

The knowledge base (which we call "kb") is the `kb` directory. It has a special structure.

For starters, it contains a directed graph of markdown files, where `kb/GOALS.md` is the entrypoint; a reader should be able to start there and navigate to whatever file they wish via internal links (described below). Thesis here is that all organization should flow from goals. Any .md file in kb that's not referenced from GOALS.md is incorrectly structured. Please be proactive about keeping the structure orderly. If you notice something is organized sub-optimally, call it out to your human and work with them to improve the strucutre.

Second: It uses legal citation patterns to document all cross-references. These will be described below in more detail, but at a high level: When you change something in the repo, check all files that reference that thing, and update them as necessary to ensure that they remain consistent with the new content. (Yes this can lead to doing recursive updates. That's fine! We want to keep kb consistent and complete.)

kb can and should reference things outside of the `kb` dir. So remember when changing any file (whether in `kb` or not), grep its path and check where its referenced to ensure everything stays consistent and complete. So if change `scripts/sync-files.js`, make sure you grep `scripts/sync-files.js`; if you change `kb/file-syncing/overview.md`, grep that path.

### kb file structure (`kb/**/*.md`)

kb files use `§`-numbered sections with inline citations:

```markdown

# Acme Services

## §1 Company Overview

### §1.1 Business Profile
Claim text with citation immediately after.
[Established: acme/docs/dd/cim.pdf[p2…p3]]

### §1.2 Financial Summary
Claim with multiple citations.
[Established: acme/20260108-investor-call["from"…"to"];
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
### Addressing

The format should be in the format `file_path[some way to ID salient info in the file]` (brackets + bracket content optional but encouraged where possible)

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

