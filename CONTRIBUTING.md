# Contributing

There's no code here to build or test — the whole project is a prompt (`routine/routine-prompt.md`)
and an HTML/CSS template (`template/seed.html`). Contributions that are useful:

- **New source entries** that you've personally verified work via WebFetch (not just "this looks
  like it should work" — actually test the URL). PRs adding a source should say what you tested and
  what it returned.
- **Fixes to broken or dead source URLs** — outlets restructure their sites, feeds get retired, bot
  protection gets added. If you find one that's stopped working, a PR removing or replacing it is
  welcome.
- **Additional Hunt & Detect platform mappings** (a Sentinel, Splunk, or Defender variant of the
  section in `docs/customization.md`) — either as an alternate section in the doc, or as your own
  fork's variant linked from an "Other platforms" section, whichever is cleaner once there's more
  than one.
- **Design refinements** that keep the existing token system intact (see `docs/customization.md`'s
  "Adjusting the design" section) rather than introducing a parallel styling approach.

Please don't submit changes that:

- Hardcode a personal watchlist, named-entity watch, or organization-specific detail into the
  shared template — those belong in your own fork/instance, not upstream. See the "Standing
  named-entity watch" pattern in `routine/routine-prompt.md` for how to keep customizations
  separable.
- Loosen any of the "Hard rules" (no fabricated URLs/IOCs/CVEs, no guessed field names in Hunt &
  Detect queries) — these exist because violating them has caused real problems in production.

Open an issue first for anything bigger than a source-list tweak, so we can agree on the approach
before you put the work in.
