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
| composition    | `1cca4fd` | finalize an active IME composition before a programmatic edit — otherwise the edit lands in the composing region and is reverted at `compositionend` (GBoard) | issue [#372] · PR [#373] (open) · repro story merged [#371] |
| selection-sync | `c711c53` | `editor.selection = …` moves the DOM caret, not just the model — fixes the rich editor caret jumping to the start on iOS refocus                              | issue [#386] · repro PR merged [#387] · fix PR TODO         |
| composition-tap | `0b287ef` | honor a tap that ends an IME composition as a deliberate caret move, instead of restoring the model selection — fixes Android markdown-toolbar bold: after typing into fresh `**|**` markers, the first tap-out snaps the caret back and only the second works (Voyager [#2292]) | issue TODO · fix PR TODO |

When a fix lands in an upstream **release**, drop its commit from this branch on
the next bump (rebasing onto the new tag auto-skips it). Hashes in the table
churn on every rebase — trust the commit **subjects** on `voyager`, and update
the table after each bump.

> **⚠️ Lesson from 2026-07:** composition-tap shipped in Voyager's
> `editate@0.6.6.patch` (Voyager [#2292]) but was silently dropped when the
> 0.6.8/0.6.9 patches were regenerated from this branch — the fix only lived on
> the orphaned `fix/honor-composition-ending-tap` branch, never on `voyager`,
> and this table didn't list it. After every bump, before committing the patch
> in Voyager, verify each row of this table survives in the built output
> (e.g. `grep -c pointerdown lib/index.js` ≥ 1 for composition-tap).

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
[#2292]: https://github.com/aeharding/voyager/pull/2292
