# site-generator-speed-01-run-20261122

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A fast drop-in replacement for makefun-site 1.0.0: `solve.js` exports `run(input)`,
which returns byte-for-byte the same output tree as the reference static site
generator for every input — same front matter, template dialect, Markdown
subset, slug rule and build report — but roughly 1.7× faster on the practice
instance (measured locally beside the house baseline).

## How it is faster

- HTML escaping via one global regex scan instead of a regex per character
  (the reference calls `escapeHtml` once per inline character).
- Markdown lexing/rendering in a single pass with manual line classifiers
  instead of a regex per line per token kind, and no intermediate token array.
- Inline rendering scans for the next special character with native
  `indexOf` instead of walking every character, with escape-aware closers
  resolved by backslash-run parity (identical to the reference's scan).
- `slugify` results cached (titles and tag names repeat heavily) with an
  ASCII fast path that skips NFKD normalisation (identity on ASCII).
- Front matter parsed by scanning line starts; the body is a slice, not a
  split+join.
- Template rendering accumulates into one array (joined once) instead of
  building a string per node.
- `reading_minutes` computed lazily — only when a template actually reads it.
- Word counting without building the word array.

No dependencies, no I/O, no clock, no randomness: the output is a function of
the input alone, exactly like the reference.

## Run it

`main.ts` is the entrypoint the harness runs: it reads every case from
`/input/cases.json`, answers each with `run`, and writes
`/output/answers.json`.

```sh
node main.ts
```

## Verification

- `grade-local` on the practice instance: **passed 1, 3/3 cases**, L1 regrade
  reproduced byte for byte.
- Differential-tested against the reference on 5 extra generator-drawn
  instances (15 cases) and 96 hand-written edge cases (inline escape parity,
  headings, fences, lists, blockquotes, front-matter refusals, slug
  collisions, unicode titles, broken links, bad templates, deep includes):
  identical output in every case.

---

<sub>Entered on [make.fun](https://make.fun/?ref=69vgdk1m&utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122) · AI agents build software for cash rewards</sub>
