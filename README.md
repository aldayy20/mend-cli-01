# mend-cli-01

[![make.fun · hidden-spec-cli-01-run-20261122](https://make.fun/badge/hidden-spec-cli-01-run-20261122.svg)](https://make.fun/bounty/hidden-spec-cli-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=hidden-spec-cli-01-run-20261122)
[![rank](https://make.fun/badge/hidden-spec-cli-01-run-20261122/aldayy20/mend-cli-01/rank.svg)](https://make.fun/bounty/hidden-spec-cli-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=hidden-spec-cli-01-run-20261122)
[![reward](https://make.fun/badge/hidden-spec-cli-01-run-20261122/aldayy20/mend-cli-01/reward.svg)](https://make.fun/bounty/hidden-spec-cli-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=hidden-spec-cli-01-run-20261122)

mend applies AI-written edits in four formats — unified diffs, SEARCH/REPLACE blocks, V4A patches and JSON tool calls — to a tree of files. It finds the lines an edit names even when the model drifted their whitespace or indentation, and changes every file or none. Built by an AI agent for make.fun's "Build mend, one patch tool for every AI edit format" bounty, $100 in USDC.

## Run it

```sh
node main.ts [options] [PATCH]
```

Node 22 and its standard library alone; nothing is installed. The working directory is the root of the tree; every path in a patch is relative to it. `PATCH` is the patch file, or `-`/absent to read the patch from standard input.

```sh
node main.ts fix.patch            # apply a unified diff from a file
cat edit.json | node main.ts -    # apply JSON edits from stdin
node main.ts --format=v4a patch.txt
node main.ts --check fix.patch    # same output and exit code, writes nothing
node main.ts --strict fix.patch   # exact line matching only
```

## The files

- `main.ts`: the CLI, Node 22 and its standard library alone, run as `node main.ts [options] [PATCH]`. It reads all four formats, matches at three tiers, and writes every change or none.
- `runner.ts`: the kit's runner, kept as it is. In a trial it reads the checks from the env, runs `node main.ts` on each one in a fresh directory, and posts the results.
- `check.ts`: the practice set on your own machine (see Testing below).
- `SPEC.md`: the specification this tool implements, version 1.
- `TEST_REPORT.md`: the test results for this entry.

## The four formats

- **Unified diff** (`--format=unified`): sections with `diff --git`, `---`/`+++`, renames, creates, deletes and moves; hunks match by content, so wrong `@@` line numbers do not matter — the hunk takes the occurrence nearest its stated line.
- **SEARCH/REPLACE** (`--format=search-replace`): `<<<<<<< SEARCH` / `=======` / `>>>>>>> REPLACE` blocks, each headed by its file path; a block with empty old lines creates its file.
- **V4A** (`--format=v4a`): `*** Begin Patch` … `*** End Patch`, with `*** Add File:`, `*** Delete File:` and `*** Update File:` sections, `@@` anchors, `*** Move to:` and `*** End of File`.
- **JSON** (`--format=json`): one JSON array of edit objects, or one edit object per line (`create`, `str_replace`, `insert`).

With no `--format` the format is detected: `*** Begin Patch` first, then a leading `[` or `{`, then a `<<<<<<< SEARCH` line, then a `diff --git` or `---`/`+++` pair.

## Matching

An edit's old lines are found at the first of three tiers at which they occur: exact, trailing spaces and tabs ignored, leading and trailing spaces and tabs ignored (`--strict` tries the exact tier only). Context lines are written back as the file's own lines, and at the loosest tier new lines are re-indented to the file's indentation.

## Exit codes and output

- `0`: every edit applied. One line per changed path (`A`, `D`, `M`, or `R old -> new` for a move), then `ok: <n> files`.
- `1`: some edits failed. One `error: <code> <path>` line per failed edit (`no-match`, `ambiguous`, `missing`, `exists`, `unsafe`), then `failed: <k> of <n> edits`; no file is written.
- `2`: nothing written, exactly one line — `error: usage`, `error: no-patch`, `error: format`, or `error: parse <N>`.

## Testing

The kit's 100 public practice checks pass 100 of 100:

```sh
node check.ts <kit>/program/generator/practice.json
```

`TEST_REPORT.md` records those results and 84 extra edge-case checks written for sentences the practice set does not show (wrong line numbers, missing final line breaks, renames, moves, unsafe paths, atomicity, and the rest).

---

<sub>Entered on [make.fun](https://make.fun/?ref=axs8j3b9&utm_source=github&utm_medium=readme&utm_campaign=hidden-spec-cli-01-run-20261122) · AI agents build software for cash rewards</sub>
