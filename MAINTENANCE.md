# Maintaining this fork

This `voyager` branch is [Voyager](https://github.com/aeharding/voyager)'s soft
fork of editate — it exists only so Voyager can ship not-yet-upstream fixes via a
pnpm patch. It tracks the pinned upstream tag plus each fix as a clean,
cherry-pickable commit, with repro stories (`stories/repro/`) and e2e. Voyager
builds this branch and diffs it against the npm-published tag to produce its
`patches/editate@<ver>.patch`. **Goal: upstream everything and delete this fork.**

Currently rebased onto **0.6.26**.

## Patches on this branch

| Fix             | Commit    | What                                                                                                                                                                                                                                                                              | Upstream                                                               |
| --------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| composition     | `1f489a1` | finalize an active IME composition before a programmatic edit — otherwise the edit lands in the composing region and is reverted at `compositionend` (GBoard)                                                                                                                     | issue [#372] · PR [#373] (open, **stale**) · repro story merged [#371] |
| composition-tap | `60b26cb` | honor a tap that ends an IME composition as a deliberate caret move, instead of restoring the model selection — fixes Android markdown-toolbar bold: after typing into fresh `**\|**` markers, the first tap-out snaps the caret back and only the second works (Voyager [#2292]) | issue TODO · fix PR TODO                                               |

When a fix lands in an upstream **release**, drop its commit from this branch on
the next bump (rebasing onto the new tag auto-skips it). Hashes in the table
churn on every rebase — trust the commit **subjects** on `voyager`, and update
the table after each bump.

**Landed upstream, dropped from this branch:**

- **selection-sync** (`editor.selection = …` moves the DOM caret) — issue [#386],
  repro PR merged [#387], fixed upstream by [#404] in **0.6.16**. Upstream's
  version is event-driven (`selectionchange` → `setTimeout(…, 50)` →
  `syncDomSelection`, tracking a `domSelection` mirror) rather than our
  synchronous write from the setter, so the DOM caret now lands **~50ms late**
  on a programmatic set. Upstream deleted the merged repro story in `a521ba6`.
- **selectionchange during composition** ([#388], in 0.6.9) — our read-only
  typeahead ask [#383]; upstream **reverted** it in [#418] (0.6.20). Voyager
  doesn't depend on it: `controller.ts` tracks composition off the native
  `compositionstart`/`compositionend`/`input` events and reads `liveDomState`.

> **⚠️ Lesson from 2026-07:** composition-tap shipped in Voyager's
> `editate@0.6.6.patch` (Voyager [#2292]) but was silently dropped when the
> 0.6.8/0.6.9 patches were regenerated from this branch — the fix only lived on
> the orphaned `fix/honor-composition-ending-tap` branch, never on `voyager`,
> and this table didn't list it. After every bump, before committing the patch
> in Voyager, verify each row of this table survives in the built output
> (e.g. `grep -c pointerdown lib/index.js` ≥ 1 for composition-tap).

> **⚠️ Lesson from 2026-08 (0.6.9 → 0.6.26):** the `apply` hook contract flipped
> upstream — `next()` with a nullish argument now **cancels** the operation
> (0.6.9 treated a bare `next()` as "continue"). The composition fix called
> `next()`, so after the rebase every edit in a mounted editor was silently
> dropped; only `npm test` caught it (`editor.window.spec.ts`). It must call
> `next(op)`. **PR [#373] still has the old `next()` and is stale against
> upstream `main`.** Run the fork's own `npm test` after every rebase — a patch
> that merely _applies_ proves nothing.

## Regenerate Voyager's patch (bump → `X`)

1. Here: `git rebase --onto <X-tag> <old-tag> voyager`, drop anything now
   upstream, then `npm run build && npm test && npm run e2e`.
2. Verify every table row survives the build (see the 2026-07 lesson). Diffing
   against a pristine build of `<X-tag>` makes this concrete:
   `grep -c pointerdown lib/index.js` (composition-tap) and the count of
   `"apply"` hook registrations (composition) must both exceed the pristine
   build's.
3. In Voyager: bump the `editate` pin, drop the old `editate@<old>` entry from
   `pnpm-workspace.yaml`, `pnpm install`, `pnpm patch editate@X`, copy this
   branch's built `lib/index.js` + `lib/index.cjs` into the edit dir,
   `pnpm patch-commit`, delete `patches/editate@<old>.patch`, then
   `prettier -w pnpm-workspace.yaml`.

> The patch is a large minified diff — the local vite build renames variables
> differently than the npm-published build, so it's not byte-for-byte (it still
> applies correctly). Review the source commits here, not the `.patch`.

[#371]: https://github.com/inokawa/editate/pull/371
[#372]: https://github.com/inokawa/editate/issues/372
[#373]: https://github.com/inokawa/editate/pull/373
[#383]: https://github.com/inokawa/editate/issues/383
[#386]: https://github.com/inokawa/editate/issues/386
[#387]: https://github.com/inokawa/editate/pull/387
[#388]: https://github.com/inokawa/editate/pull/388
[#404]: https://github.com/inokawa/editate/pull/404
[#418]: https://github.com/inokawa/editate/pull/418
[#2292]: https://github.com/aeharding/voyager/pull/2292
