# Knowledge base (kb)

The knowledge base (which we call "kb") is the `kb` directory. It has a special structure.

For starters, it contains a directed graph of markdown files, where `kb/GOALS.md` is the entrypoint. A reader should be able to start there and navigate to whatever information is relevant to what they're trying to accomplish via descriptions and internal links (described below). Thesis here is that all organization should flow from goals. 

(Any .md file in kb that's not referenced from GOALS.md is incorrectly structured. Please be proactive about keeping the structure orderly. If you notice something is organized sub-optimally, call it out to your human and work with them to improve the strucutre.)

Second: it uses legal citation patterns to ensure knowledge and cross-references stay consistent, current, and valid.


## File structure (`kb/**/*.md`)

kb files use `§`-numbered sections with inline citations:

```markdown
# Topic Name

## §1 First Area

### §1.1 Key Finding
The claim you're documenting goes here.
[Established: path/to/source-file.pdf[p2…p3]]

### §1.2 Related Finding
A claim supported by multiple sources, with cross-references.
[Established: path/to/another-source.pdf[p5…p8];
cf. kb/related-topic/subtopic§2;
but see: path/to/contradicting-source["The exact quote"…"that ends here"]]

## §2 Second Area

### §2.1 Sub-finding
A claim that connects to other parts of the knowledge base.
[Established: path/to/primary-source["Start of quote"…"end of quote"];
accord: kb/other-topic/detail§2.1;
see generally: kb/frameworks/some-framework§3]
```

Rules:

- Subsections nest: `§2` → `§2.1` → `§2.1.1` (depth as needed).
- **Every factual claim must have a citation! No exceptions.** We're working towards having all call transcripts, relevant screenshots, etc. stored in the repo. Until we reach that point, please not missing citations with `[citation needed]`.

### Addressing

The format should be in the format `file_path[some way to ID salient info in the file]` (brackets + bracket content optional but encouraged where possible)

| Type | Format | Example |
|------|--------|---------|
| Section | `kb-path§section` | `kb/topic/subtopic§2.1` |
| Whole source | `path/to/file.ext` | `docs/report.pdf` |
| Text span | `source["from"…"to"]` | `docs/report.pdf["Revenue grew"…"over 5 years"]` |
| Page range | `source[pN…pM]` | `docs/report.pdf[p12…p14]` |
| Historical | `source@commit["from"…"to"]` | `docs/report.pdf@a3f7c2["from"…"to"]` |

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

## Your kb responsibilities

When you change something in the repo, check all files that reference that thing, and update them as necessary to ensure that they remain consistent with the new content. (Yes this can lead to doing recursive updates. That's fine! We want to keep kb consistent and complete.)

kb can and should reference things outside of the `kb` dir. So remember when changing any file (whether in `kb` or not), grep its path and check where its referenced to ensure everything stays consistent and complete. So if change `scripts/sync-files.js`, make sure you grep `scripts/sync-files.js`; if you change `kb/file-syncing/overview.md`, grep that path.
