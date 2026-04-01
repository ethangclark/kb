# Ideas

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
