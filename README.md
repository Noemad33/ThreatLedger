# ThreatLedger

A self-updating daily cybersecurity + financial-sector threat brief, published as a Claude
Artifact and kept current by a scheduled Claude Code routine — no server, no cron box, no
infrastructure to maintain. Every morning it re-reads a fixed list of primary sources (vendor IOC
repos, CISA's KEV catalog, abuse.ch, ransomware.live, independent-researcher discovery searches,
and more), writes a fresh edition, and republishes the same page in place.

It ships with two tabs:

- **Daily Brief** — an actionable IOC block up top (defanged indicators ready for a blocklist),
  an executive summary, critical vulnerabilities, active campaigns, a dedicated Identity & Cloud
  Access section (AiTM phishing, token theft, conditional-access bypass, cloud IAM/storage
  misconfiguration), a financial-sector watch section, social-media pulse, a persistent
  researcher/threat-actor continuity tracker (so it can say "this is the third vendor this
  identity has hit in three weeks" instead of treating every morning as history-free), and full
  source citations.
- **Hunt & Detect** — for findings with concrete technical indicators, whichever of a CrowdStrike
  Falcon Advanced Event Search (LogScale/CQL) query, an Elastic Security detection rule (query +
  severity/risk-score/MITRE ATT&CK/index metadata), and a Microsoft Defender/Sentinel Advanced
  Hunting KQL query genuinely apply — built from that day's actual IOCs and attack chains, not
  generic examples, and covering only the platforms a given finding is actually visible to. For
  AWS/Azure/cloud-identity findings, it checks published rules from Elastic's own
  [`detection-rules`](https://github.com/elastic/detection-rules) repo, [SigmaHQ](https://github.com/SigmaHQ/sigma),
  and Microsoft's [Azure-Sentinel](https://github.com/Azure/Azure-Sentinel) repo first, and cites
  the real rule when one exists instead of hand-authoring from scratch.

The shipped configuration is tuned to one example stack — Azure + AWS, Elastic, CrowdStrike
(EDR/NGSIEM/Cloud Security), and Microsoft Entra/Defender — as a complete, working reference to
edit from. See [Customizing](#customizing) for swapping in your own.

This repo is a **template**, not a running service. You provide your own Claude Artifact and your
own scheduled routine (both free with a Claude account); this repo gives you the prompt and the
HTML/CSS design system that make it work, plus the reasoning behind every non-obvious decision in
them, so you can adjust it to your own sources, sectors, and tooling.

## How it works

1. **The page** is a single static HTML file — no backend, no build step — with a small amount of
   vanilla JS for the tab switcher. It's published as a private [Claude
   Artifact](https://claude.ai) (a hosted page URL). If you use Claude Artifacts' database
   capability, the page can also carry a tiny persistent store for the continuity tracker.
2. **The routine** is a scheduled Claude Code cloud session (a "routine" — see [Claude Code's
   scheduling docs](https://docs.claude.com/en/docs/claude-code)) that fires on a cron schedule,
   researches the last 24–72 hours of threat activity across a defined source list, and republishes
   the artifact in place with a new edition.
3. Every morning: read the current edition (for the volume number and continuity data) → research
   → write the new HTML → publish it to the *same* artifact URL → optionally update the continuity
   tracker.

Nothing here requires paid infrastructure beyond a Claude account with routines and artifacts.

## Quickstart

1. **Publish the seed page.** Open Claude Code (or claude.ai), and ask it to publish
   [`template/seed.html`](template/seed.html) as a new Artifact. Note the resulting artifact URL —
   you'll need it in the next step. (Private artifacts are fine; only your routine needs to reach it.)
2. **Fill in the routine prompt.** Open [`routine/routine-prompt.md`](routine/routine-prompt.md)
   and replace `[YOUR_ARTIFACT_URL]` with the URL from step 1. Read through the
   [Customization](docs/customization.md) doc now if you want to change the source list, the
   watched X/Twitter accounts, or add a standing named-entity watch (e.g. "alert me the moment
   *my company* shows up on a leak site") — easier to do before the first run than after.
3. **Create the scheduled routine.** In Claude Code, use the `/schedule` command (or ask Claude to
   set up a scheduled routine) with the finalized prompt and a cron expression for the time you
   want it to run (the original deployment uses `30 12 * * *`, 12:30 UTC daily). Point it at the
   artifact from step 1.
4. **Let it run once**, then open the artifact URL. That's Vol. 1.

From then on it updates itself every day on schedule. No further action needed unless you want to
tune what it covers.

## Design system

Newsreader (headings) + IBM Plex Sans (body) + IBM Plex Mono (data/code), all loaded from Google
Fonts. Warm-paper light mode / near-black dark mode, a brass accent color, and three semantic
severity colors (critical/high/watch) kept separate from the accent. The full token set lives at
the top of the `<style>` block in both `template/seed.html` and `routine/routine-prompt.md` — change
the CSS custom properties there to reskin it; the rest of the layout follows from the tokens.

## Customizing

See [`docs/customization.md`](docs/customization.md) for a walkthrough of the parts you're most
likely to want to change: the source list, the watchlist, adding a standing named-entity watch,
swapping in a different EDR/SIEM for the Hunt & Detect tab, and adjusting the design tokens.

## A note on trust

This is an AI research pipeline that fetches, summarizes, and republishes security content
automatically and without a human in the loop each morning. It's built with hard rules against
fabricating URLs, IOCs, or CVE details, and it's meant to say "nothing new" honestly rather than
invent content — but it is still AI output. Treat every IOC, query, and detection rule as a
starting point to verify against its cited primary source before enforcing it in production,
exactly as the page's own colophon says on every edition.

## License

MIT — see [`LICENSE`](LICENSE). Use it, fork it, change it, ship your own version.
