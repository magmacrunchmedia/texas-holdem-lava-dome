# Texas Hold'Em Lava Dome — agent brief

One game, three versions, one repo:

- `web/` — browser version (adenosine engine, plain JS). Source of truth for
  rules and tuning (`js/config.js`, `js/dome.js`). Deployed by the website
  repo: run `make sync-texas-holdem-lava-dome` there to copy `web/` into
  `arcade/solitaire_THLD/` (historical folder name, kept for the URL). Never
  edit the website repo's copy directly — it gets overwritten.
- `wii/` — Wii port (magnolia engine, C99). Has its own `AGENTS.md` and a
  README that records the deliberate differences from the web build (ace-high
  restamping, raise-buys-a-card). Expects magnolia checked out beside this
  repo.

- `tui/` — terminal version (the `magmacrunch.engine` TUI engine, Python).
  `python -m lavadome`. Has its own `README.md`, which records the deliberate
  differences it shares with the Wii port.

A rules change is not done until all three versions have it (or the commit says
why one is skipped).

## Cache-buster stamps in `web/index.html`

Every `?v=` in `web/index.html` is the first eight hex of SHA-256 over the file
it stamps, with newlines normalised to LF. Get one wrong and visitors keep
serving the cached old bytes, so the change reaches nobody and the page still
loads.

`../shared/adenosine-cards.js` and `../shared/adenosine-chat.js` are the ones
to watch: they name files that do not exist in this repo at all. They resolve
only once `web/` has been copied into the website's `arcade/`, and they go
stale when *that* repo updates the shared bundles — which nothing here can
notice.

The website's `.githooks/pre-commit` recomputes stale stamps in its copy of
this page, but that repair never travels back here, and
`make sync-texas-holdem-lava-dome` copies `web/` over `arcade/solitaire_THLD/`
verbatim. So a stamp corrected there is reverted by the next sync, silently,
on a game nobody touched. **The fix belongs in this repo.** Recompute one from
a website checkout, and check the whole site after any sync:

```
node scripts/check-cache-busters.mjs --digest arcade/shared/adenosine-cards.js
npm run check:cachebust
```

## Checking the rules

The web build has no test suite, so unlike George Boole there is no assertion
table to agree with. `tui/` supplies one for the part that matters most:
`tui/tools/js_oracle.mjs` loads the shipped `arcade/shared/adenosine-cards.js`
in node, and `tui/tests/test_handeval.py` runs the Python evaluator against it
over thousands of random hands, comparing name, rank, points, tiebreakers and
description. Same method `web/js/config.js` records using when AdCards replaced
this game's original evaluator.

If you change hand evaluation anywhere, run that.

## `tui/LICENSE` and `tui/NOTICE` are copies

The originals are at the repo root, where they cover `web/` and `wii/` too.
The copies exist because the wheel is built from `tui/` and PolyForm requires
the notice to travel with the distribution — a wheel built from a subdirectory
cannot reach a file above it, and `force-include` does not help because
`python -m build` builds the wheel from an unpacked sdist that has no parent.

**Relicensing means changing all three copies**, here and in
`george-boole/tui/`. Nothing checks this automatically.
