# Knowledge base (kb)

The `kb` directory is a structured, citation-backed knowledge base. Your job is to keep it consistent and complete.

## Your responsibilities

**When you change anything in the repo**, grep for its path and update every reference to keep kb consistent. This applies to files both inside and outside `kb`. Recursive updates are expected.

**Keep the graph connected.** `kb/GOALS.md` is the entrypoint — every `.md` file in `kb` must be reachable from it via links. All organization flows from goals. If you notice an orphaned or poorly organized file, flag it and work with the user to fix it.

**Every factual claim must have a citation.** No exceptions. If a source isn't available yet, mark it `[citation needed]`.

## File structure

kb files use `§`-numbered sections with inline citations. Subsections nest: `§2` → `§2.1` → `§2.1.1`.

```markdown
# Topic Name

## §1 First Area

### §1.1 Key Finding
The claim you're documenting goes here.
[Established: path/to/source.pdf[p2…p3]]

### §1.2 Related Finding
A claim with multiple sources and cross-references.
[Established: path/to/another-source.pdf[p5…p8];
cf. kb/related-topic/subtopic§2;
but see: path/to/contradicting-source["The exact quote"…"that ends here"]]
```

## Addressing

References use the format `file_path[locator]` (locator is optional but encouraged):

| Type | Format | Example |
|------|--------|---------|
| Section | `kb-path§section` | `kb/topic/subtopic§2.1` |
| Whole source | `path/to/file.ext` | `docs/report.pdf` |
| Text span | `source["from"…"to"]` | `docs/report.pdf["Revenue grew"…"over 5 years"]` |
| Page range | `source[pN…pM]` | `docs/report.pdf[p12…p14]` |

For pinning a reference to a specific commit: `source@commit["from"…"to"]` (e.g. `docs/report.pdf@a3f7c2["from"…"to"]`).

## Citation signals

| Signal | Meaning |
|--------|---------|
| `Established:` | Primary source for this claim |
| `Supersedes:` | Replaces a prior claim entirely |
| `Cf.` | Compare with (related but different angle) |
| `But see:` | Contradictory or qualifying information |
| `Accord:` | Corroborated by another source |
| `See generally:` | Background or framework reference |
