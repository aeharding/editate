# Maintaining this fork

This `voyager` branch is [Voyager](https://github.com/aeharding/voyager)'s soft
fork of editate — it exists only so Voyager can ship not-yet-upstream fixes via a
pnpm patch. It tracks the pinned upstream tag plus each fix as a clean,
cherry-pickable commit, with repro stories (`stories/repro/`) and e2e. Voyager
builds this branch and diffs it against the npm-published tag to produce its
`patches/editate@<ver>.patch`. **Goal: upstream everything and delete this fork.**

## Patches on this branch

| Fix            | Commit    | What                                                                                                                                                           | Upstream                                                    |
| -------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| composition    | `fb286b4` | finalize an active IME composition before a programmatic edit — otherwise the edit lands in the composing region and is reverted at `compositionend` (GBoard) | issue [#372] · PR [#373] (open) · repro story merged [#371] |
| selection-sync | `06ea47d` | `editor.selection = …` moves the DOM caret, not just the model — fixes the rich editor caret jumping to the start on iOS refocus                              | issue [#386] · repro PR [#387] (open) · fix PR TODO         |

When a fix lands in an upstream **release**, drop its commit from this branch on
the next bump (rebasing onto the new tag auto-skips it).

## Regenerate Voyager's patch (bump → `X`)

1. Here: `git rebase --onto <X-tag> <old-tag> voyager`, drop anything now
   upstream, then `npm run build && npm test && npm run e2e`.
2. In Voyager: bump the `editate` pin, `pnpm patch editate@X`, copy this branch's
   built `lib/index.js` + `lib/index.cjs` into the edit dir, `pnpm patch-commit`,
   then `prettier -w pnpm-workspace.yaml`.

> The patch is a large minified diff — the local vite build renames variables
> differently than the npm-published build, so it's not byte-for-byte (it still
> applies correctly). Review the source commits here, not the `.patch`.

[#371]: https://github.com/inokawa/editate/pull/371
[#372]: https://github.com/inokawa/editate/issues/372
[#373]: https://github.com/inokawa/editate/pull/373
[#386]: https://github.com/inokawa/editate/issues/386
[#387]: https://github.com/inokawa/editate/pull/387
