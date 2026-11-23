# TEST_REPORT.md — mend-cli-01

Entry for the make.fun bounty "Build mend, one patch tool for every AI edit format"
(`hidden-spec-cli-01-run-20261122`). Built 2026-11-23. The CLI is `main.ts`
(Node 22, standard library alone); `SPEC.md` in this repo is the specification it
implements.

## Results

| Suite | Result |
|---|---|
| Kit practice set (`node check.ts <kit>/program/generator/practice.json`) | **100 of 100 passed, practice score 1000** (unified 30/30, search_replace 20/20, v4a 20/20, json 15/15, cli 15/15) |
| Kit trial, `grade-local --driver process` (practice trial 0, via `runner.ts` and the `MAKEFUN_ENV_URL` hand-over) | **100 of 100 checks, score 1000**; L1 regrade reproduced byte for byte; replay check passed; 0 model calls, $0.00 spend |
| Extra edge-case suite written for this entry (85 cases) | **85 of 85 passed** |

The kit starter scored 24/100 on the same practice set before this work
(it had no unified, SEARCH/REPLACE or V4A reader); the house entry is a
strict (tier-0-only) applier, so the fuzzy three-tier matching is what has to
beat it.

## Practice set

Run from this directory against the unpacked kit:

```sh
node check.ts <kit>/program/generator/practice.json
# 100 of 100 passed: practice score 1000
```

Families covered: `u-exact`, `u-offset`, `u-nearest`, `u-trailing-ws`,
`u-indent`, `u-crlf`, `u-noeol`, `u-insert`, `u-create-delete`, `u-rename`,
`u-multi-file`, `u-missing`, `u-nomatch`, `u-parse`, `u-fenced-bare`,
`s-basic`, `s-create`, `s-ambiguous`, `s-missing`, `s-nomatch`, `s-parse`,
`s-whitespace`, `s-crlf`, `s-multi-block`, `s-multi-file`, `s-delete-lines`,
`v-add-delete`, `v-update-plain`, `v-update-anchor`, `v-cursor`, `v-eof`,
`v-move`, `v-exists-missing`, `v-nomatch`, `v-parse`, `v-crlf`, `v-indent`,
`j-array`, `j-create`, `j-str-replace`, `j-insert`, `j-crlf`, `j-ambiguous`,
`j-faults`, `j-parse`, `c-usage`, `c-format`, `c-format-error`, `c-check`,
`c-check-fail`, `c-strict`, `c-no-patch`, `c-unsafe`, `c-noop`.

## Extra edge-case suite (85 cases)

A separate harness run during development (kept out of the repo) exercised
spec sentences the practice set does not show, one run of `node main.ts` per
case in a fresh directory, checking exit code, stdout and every resulting
file. Final re-run after the last `main.ts` change: practice 100/100 and
extra 85/85. Groups and what they pin down:

- **Unified parsing**: `---` without `+++` inside a `diff --git` header
  (`error: parse` at the line after it); malformed `diff --git` and `@@`
  lines; a hunk before any section; a lone `---` with no section started is
  prose, so a following `@@` is a hunk before any section.
- **Unified edits**: creation via `/dev/null` and via `new file mode`;
  deletion via `/dev/null` and via `deleted file mode`, including a deletion
  whose hunks do not match (hunks are not read); rename from/to with and
  without hunks; move to an existing path (`exists`) and from a missing path
  (`missing`); `---`/`+++` paths with a tab timestamp; `a/`/`b/` stripping;
  unsafe `a/../evil.txt` reported as `../evil.txt`; prose and code fences
  around a patch; a section with no hunks (no-op); a creation with no `+`
  lines (empty file).
- **Unified hunks**: a hunk header with text attached directly after the
  closing `@@`; empty-`O` insertion at the top, in the middle and at the
  end; insertion at the end of a file with no final line break, with and
  without `\ No newline at end of file`; `\` after a `-` line only (final
  break kept); multi-hunk line-number shift (`d`); nearest occurrence with a
  tie going to the earlier one; CRLF files kept CRLF.
- **SEARCH/REPLACE**: a block taking the previous block's path; `ambiguous`
  when old lines occur twice; creation with empty old and new lines; creation
  over an existing file (`exists`); missing `>>>>>>> REPLACE` and first block
  with no path (`error: parse` at the SEARCH line); trailing whitespace on
  markers; tier-2 re-indentation (a new line with no indent in the patch
  takes the file's indent); deleting lines with empty new lines.
- **V4A**: first non-blank line not `*** Begin Patch`, no non-blank lines,
  last non-blank line not `*** End Patch` (`error: parse` at lines + 1);
  Add with blank lines inside and trailing blanks stripped; Delete with lines
  (`error: parse`); Update with no chunk and no Move to (`error: parse` at
  the Update line); Update with only Move to (pure move, `R` line); anchor
  not found (`no-match`); anchor matched at tier 1; end-of-file chunk whose
  old lines do not end at the last line (`no-match`); a body line after
  `*** End of File` without `@@` (`error: parse`); Move to an existing path
  (`exists`); Add over an existing file (`exists`); CRLF file kept CRLF;
  two anchored chunks applied in cursor order.
- **JSON / CLI**: array-form fault reported at the array's start line; bad
  JSON Lines line reported at its line; wrong command; `insert` out of range
  (`no-match`); `create` writes `file_text` byte for byte (no newline
  added); two positionals, duplicated `--check`, bad `--format`, unknown
  flag (`error: usage`); `--` ending the options; `--format=json` forced
  onto a unified patch; empty patch and prose-only patch with auto detection
  (`error: format`); missing patch file (`error: no-patch`); multiple
  failures reported in patch order with later edits still running; atomicity
  (a creating edit followed by a failing edit writes nothing); output rows
  sorted by path, `R` rows sorted by the new path; nested directory creation;
  `--check` changing nothing; `ok: 0 files` for a no-op.

## Judgement calls (where SPEC.md leaves room)

These follow the most literal reading of the spec; the practice set confirms
several of them (noted where it does):

- A `--- ` line with no following `+++ ` is a parse error only inside an
  already-started section's header; on its own it is prose.
- A V4A body line after `*** End of File` starts no new chunk without `@@`.
- At tier 2, a new line whose old first line had no indent takes the file's
  indent (empty prefix replaced by `F`) — confirmed by the practice
  `v-indent` case, where `} // node` becomes `  } // node`.
- An empty unified hunk inserted at the end of a file makes the result end
  with a line break unless a `\` line follows its last context or `+` line.
- A `\` line after a `-` line only does not remove the final line break.

## Environment notes

- Local checks ran on Node v24.22.1 (this machine); the grader runs Node
  22.23.4, which runs this TypeScript as-is. `main.ts` uses only erasable
  TypeScript and the standard library, the same constructs the kit starter
  uses.
- `grade-local` was run with `--driver process` (no Docker sandbox here);
  the server grades in its Docker sandbox, which the process run notes.
- Repo snapshot: 7 files, 65,234 bytes total, largest file `main.ts`
  (28,540 bytes) — far under the 1 MiB per file / 4 MiB total caps.
- The kit itself (`hidden-spec-cli-01-run-20261122-kit/`, 9.5 MB) is kept
  outside this directory and is not part of the repo.
