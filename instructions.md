# Knowledge base (kb)

The `kb` directory is a structured, citation-backed knowledge base.

## When this applies

This information is relevant when you want to search the knowledge base for information, or when the user asks you to create or modify kb entries. Maintaining the knowledge base is not a background responsibility — don't let it compete for attention when you're focused on other work.

## Principles

**Graph connectivity.** `kb/GOALS.md` is the entrypoint. Every `.md` file in `kb` must be reachable from it via links. All organization flows from goals. If you notice an orphaned or poorly organized file, flag it.

**Keep files concise.** Each kb file should be a 1- or 2-pager — like keeping modules appropriately sized in code. If a file is growing beyond that, split it.

**Every factual claim must have a citation.** No exceptions. If a source isn't available yet, mark it `[citation needed]`.

**Consistency on edit.** When you modify a kb file, grep for its path and update every reference so cross-links stay valid. This applies to references both inside and outside `kb`. If updating a reference triggers further updates, follow the chain.

## File structure

kb files use standard markdown headings. Cross-references use heading anchors (`kb/path/to/file.md#heading-name`). Inline citations follow each claim.

```markdown
# Topic Name

## First Area

### Key Finding
The claim you're documenting goes here.
[Established: path/to/source.pdf[p2…p3]]

### Related Finding
A claim with multiple sources and cross-references.
[Established: path/to/another-source.pdf[p5…p8];
cf. kb/related-topic/subtopic.md#some-heading;
but see: path/to/contradicting-source["The exact quote"…"that ends here"]]
```

## Addressing

References use the format `file_path[locator]` (locator is optional but encouraged):

| Type | Format | Example |
|------|--------|---------|
| Section | `kb/path/file.md#heading` | `kb/topic/subtopic.md#key-finding` |
| Whole source | `path/to/file.ext` | `docs/report.pdf` |
| Text span | `source["from"…"to"]` | `docs/report.pdf["Revenue grew"…"over 5 years"]` |
| Page range | `source[pN…pM]` | `docs/report.pdf[p12…p14]` |

For pinning a reference to a specific commit: `source@commit["from"…"to"]` (e.g. `docs/report.pdf@a3f7c2["from"…"to"]`).

## Citation signals

Use `Established:` as the default. The others are optional precision for when the relationship matters:

| Signal | Meaning |
|--------|---------|
| `Established:` | Primary source for this claim |
| `Supersedes:` | Replaces a prior claim entirely |
| `Cf.` | Compare with (related but different angle) |
| `But see:` | Contradictory or qualifying information |
| `Accord:` | Corroborated by another source |
| `See generally:` | Background or framework reference |

---

# TODO

## Including source material in repo as a matter of course

Call transcripts, relevant screenshots, etc. should be included. The process by which this occurs should be highly automated.

## Pre-merge validation script

Build a validation script (runnable locally and in CI) that checks kb integrity before merge. The goal is structural correctness enforced automatically — like type-checking in a TypeScript repo — so that consistency doesn't depend on manual diligence.

The script should check:

1. **Graph connectivity**: Every `.md` in `kb` is reachable from `kb/GOALS.md` via internal links. Flag orphaned files.
2. **Reference validity**: Every cross-reference (`kb/path/file.md#heading`) resolves to an existing file and heading. Flag broken links.
3. **Citation coverage**: Every heading that contains prose also contains a citation bracket (`[Established: ...]`, `[citation needed]`, etc.). Flag uncited claims.
4. **File size**: Flag any kb file exceeding ~2 pages (rough heuristic: e.g. >120 lines) as a candidate for splitting.

This should run as a pre-merge check so that kb drift is caught structurally rather than relying on the person (or agent) editing to remember all the rules. The current manual process is a pilot — this automation is the intended end state.
