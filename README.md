# site-generator-speed-01 — fast drop-in for makefun-site 1.0.0

[![make.fun](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/thomasmoa97/site-generator-speed-01/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/thomasmoa97/site-generator-speed-01/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own `makefun-site` 1.0.0: `solve.js` returns
exactly the output tree `makefun-site`'s `build` returns for every case input —
each page's `index.html`, the index and tag pages, `build-report.txt` and each
static file, byte for byte — but in about a third of the CPU time.

## How it is faster

The pipeline is reimplemented from scratch against the reference semantics
(front matter, template dialect, Markdown subset, slug rule, report), with the
hot paths rewritten:

- **Bulk HTML escaping** — plain runs are escaped in one native `replace`
  instead of one regex call per character.
- **Index-based inline scanning** — the Markdown inline renderer finds the
  next markup character with a regex scan instead of walking every character
  in JavaScript, and renders nested spans by index ranges (no substrings).
- **Slug cache** — `slugify` results are cached per build; tag and heading
  slugs repeat heavily.
- **Fused Markdown lex + render** — no token objects; blocks render straight
  into an output array.
- **Front matter off the line array** — the source is split once and the same
  lines feed the front-matter parser, the word counter and the Markdown
  renderer (no re-join / re-split of the body).
- **Array-based template rendering** — templates render into an array joined
  once, with a push/pop scope stack instead of a scope object per loop
  iteration.
- `main.ts` is the house driver, unchanged (stdlib-only, `--no-incremental-marking`).

No dependencies, no WebAssembly, no native addon, no prebuilt binary.

## Verification (local, Node v24.22.1)

- Practice set: **3/3 cases identical** to `kit/practice/reference/expected.json`
  (compared as JSON values, exactly like the scorer).
- **90/90 extra generated cases identical** to the house baseline, drawn with
  the kit's own generator (`program/generator/subject/sites.mjs`) from 30
  fresh seeds.
- **Determinism**: two runs produce byte-identical `answers.json`.
- Solve-only time on the practice set (3 cases, 250/340/480 Markdown sources):
  baseline median **477 ms**, this entry median **~167 ms** (**~2.9x**).
  End to end through `main.ts` (including the JSON parse/stringify both
  entries share): baseline ~0.96 s wall, this entry ~0.56 s wall.
