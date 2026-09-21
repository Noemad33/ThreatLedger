# Customizing ThreatLedger

Everything the routine does lives in one file, [`routine/routine-prompt.md`](../routine/routine-prompt.md).
There's no code to change — just edit the prompt text and the next scheduled run picks it up.

## Changing the source list

The `## Sources to check` section lists every feed the routine reads, grouped by category
(structured IOC feeds, malware/IOC feeds, ransomware leak-site tracking, general infosec,
vulnerabilities, financial sector, Reddit, X/Twitter). To add a source:

- **RSS/Atom feed**: add one line with the outlet name and feed URL to the relevant category. The
  routine already knows to WebFetch a feed URL and fall back to WebSearch if the fetch fails.
- **A site with no feed**: add it as a WebSearch instruction instead (see the "Independent research
  discovery" section for examples of search-query-based sourcing).
- **A JSON/CSV API**: add the exact endpoint URL and a one-line note on what it returns, the way
  the abuse.ch and ransomware.live entries are written. Test the URL yourself first — some sites
  (URLhaus and ThreatFox's `/browse/` pages, for instance) are bot-check-gated and return nothing
  useful via WebFetch; their raw CSV/JSON export endpoints work fine. Don't assume an endpoint
  works — verify it before adding it to the prompt, the same rule the routine itself follows for
  every URL it cites.

To remove a source, just delete its line. Nothing else references the source list by name, so
there's no cleanup needed elsewhere.

## Changing the watched X/Twitter accounts

The `**X / Twitter**` section and the page's "Accounts worth following" watchlist block (near the
bottom of the `## Template` HTML) are separate — the routine *searches* for commentary from the
first list, and *displays* the second list verbatim every day (it's static content, not
regenerated). Edit both if you change who you're tracking, or they'll drift out of sync.

## Adding a standing named-entity watch

If you want the routine to check on a specific company, client, or organization every single day
regardless of what else is happening, look for the `**Standing named-entity watch (OPTIONAL...`
bullet under the ransomware.live entry. Uncomment/fill it in with your organization's name, e.g.:

```
**Standing named-entity watch:** search https://api.ransomware.live/v2/searchvictims/acme%20corp
for "Acme Corp". Note the most recent result (or "nothing on record") so you can compare day to
day; if a NEW entry appears since the last edition, treat it as a lead item regardless of what
else is happening that day.
```

You can add more than one — just repeat the pattern for each entity. Each one costs a single extra
API call per run, so there's no real limit worth worrying about.

## Matching the prompt to your own stack

The shipped prompt is a fully worked example tuned to one specific stack: Azure + AWS for cloud,
Elastic for SIEM, CrowdStrike across EDR/NGSIEM/Cloud Security, and Microsoft Entra/Defender for
identity and endpoint. It's deliberately left concrete rather than abstracted into placeholders,
so you have a complete working example to edit from rather than a fill-in-the-blanks template with
nothing to copy. Three places carry stack-specific detail — change all three together, or the page
will describe a stack it isn't actually covering:

1. **The "Reader's stack" line** near the top of the prompt (`## The deliverable` section) — one
   sentence that tells the routine which vendors deserve a slight inclusion-priority bump. Swap in
   your own cloud/SIEM/EDR/identity vendors.
2. **The "Cloud provider security feeds" and "Identity & Cloud Access discovery" source blocks** —
   built around Azure/AWS and Entra specifically. If your org runs GCP instead of (or alongside)
   Azure/AWS, add a GCP Security Bulletins / Google Cloud blog entry following the same pattern;
   if your identity provider is Okta or Ping instead of Entra, swap the Entra-specific search
   queries for your provider's equivalent terminology (Okta calls the same AiTM/token-theft
   category differently in places).
3. **The Hunt & Detect platform list** (below).

## Swapping the Hunt & Detect platforms

The tab is built around three platforms — CrowdStrike Falcon (Advanced Event Search / LogScale
query language), Elastic Security (KQL detection rules), and Microsoft Defender/Sentinel (KQL over
Advanced Hunting tables) — because that's what the original deployment runs. If you use different
tooling — Splunk (SPL), Google SecOps/Chronicle (YARA-L), Sentinel-only without Defender (plain Log
Analytics KQL, a different table/column set than Advanced Hunting) — edit the `## Hunt & Detect tab
— building it correctly` section:

- Replace the platform field/table-name list(s) with your platform's real field/table names — one
  bullet per platform, matching the pattern already there for CrowdStrike/Elastic/Defender.
- Replace `query-label` text in the template (e.g. "CrowdStrike Falcon · Advanced Event Search")
  with your platform's name, and update the hunt-card template comment block to show your platform
  count (two, three, four — however many you actually run).
- Keep the core rule intact: **only build a card when there's a concrete indicator to hunt on**,
  **never fabricate a field or table name** — if you're not sure a field exists, say so or use a
  more generic one you're confident about — and **match the query to what the platform can
  actually see** (an EDR sensor has no visibility into a router; a posture/CSPM tool doesn't speak
  in LogScale/KQL event queries, it speaks in policy names — see the cloud-misconfiguration
  guidance already in that section for the pattern). A wrong field name in a query someone might
  actually deploy is a worse failure than an honest gap.
- **CrowdStrike syntax is the one most likely to go wrong.** CrowdStrike Query Language (CQL/LogScale) looks like SQL or Splunk SPL but isn't: SQL-style `IN (...)` and Splunk-style `append [search ...]` are rejected by the Falcon console. The prompt carries an explicit CQL rules block (use `in(field=..., values=[...])` or `or`, write separate queries instead of `append`, only real `event_simpleName` values) added after a real "Expected an expression" failure. If you add another query language, add an equivalent "what this dialect is NOT" block for it.
- Only running one or two platforms? Delete the others' bullets and panel examples rather than
  leaving unused platforms in the prompt — an unused platform in the instructions is just noise
  the model has to read past every single day.

## Citing real published detection rules instead of hand-authored ones

For AWS/Azure/cloud-identity findings, the prompt doesn't just invent a query from scratch — it
first checks whether a real detection-engineering team has already published a matching rule, in:

- [`elastic/detection-rules`](https://github.com/elastic/detection-rules) — Elastic's own rules
  (`rules/integrations/aws`, `rules/integrations/azure`)
- [`SigmaHQ/sigma`](https://github.com/SigmaHQ/sigma) — vendor-neutral community rules
  (`rules/cloud/aws/cloudtrail`, `rules/cloud/azure/*`)
- [`Azure/Azure-Sentinel`](https://github.com/Azure/Azure-Sentinel) — Microsoft's own KQL analytics
  rules (`Detections/AWSCloudTrail`, `Detections/AzureActivity`, `Detections/SigninLogs`, etc.)

The lookup process (fetch a subfolder's file listing via the GitHub contents API, scan filenames
for a match, fetch and actually read the raw file before citing it, credit it with a link if it's
a genuine match) is fully spelled out in the `## Hunt & Detect tab — building it correctly`
section of the prompt — search for "check for a published rule before hand-authoring one." If you
add your own platform (see above), consider whether an equivalent rule repo exists for it (Splunk's
[`splunk/security_content`](https://github.com/splunk/security_content) is the closest analogue)
and wire it into this same lookup pattern rather than leaving your new platform hand-authored-only.

Two other source blocks feed this same goal from the news side rather than the reference side —
**Cloud provider security feeds** (AWS/Azure vendor advisories) and **Cloud-native threat research**
(Permiso, Datadog Security Labs, Wiz, Sysdig, Invictus-IR) — both already in the prompt's Sources
list. Swap these for your own cloud provider(s) and preferred research blogs the same way you'd
edit any other source entry.

## Adjusting the design

The full token set is the first rule block inside the `<style>` tag, both in `template/seed.html`
and inside the `## Template` section of `routine/routine-prompt.md`. Change the CSS custom
properties there (`--bg`, `--accent`, the three `--sev-*` pairs, etc.) — the rest of the layout
references them, so a token change reskins the whole page. **Change both files identically**, or
your seed page and your first real edition will look different.

If you reskin it, also update the `## Hard rules` line that says "Keep the visual design... IDENTICAL
to the template below" only refers to *day-to-day* consistency (so the routine doesn't redesign the
page on its own) — it doesn't stop you from deliberately changing the template once, yourself.

## Schedule and timezone drift

Cron schedules on most platforms run in UTC. If you want the brief to land at a consistent local
time, remember that a fixed UTC cron expression will drift by an hour across DST transitions in
timezones that observe it — either accept the drift (harmless, just shifts by an hour twice a
year) or adjust the cron expression manually around DST changes.

## The one gotcha worth knowing before you turn this on

If you use the Artifact database capability for the continuity tracker (the `db` capability,
`tracking/identities/entries` collection), **writes to it (`write_db`) trigger an interactive
permission prompt that nobody is present to approve in an unattended scheduled run** — the session
hangs waiting for an approval that never comes. The prompt already handles this correctly (writes
happen *after* the page publish, and a stalled write is treated as expected/harmless), but if you
restructure the routine, keep the publish call ahead of any `write_db` call in execution order. See
the comment block in `routine/routine-prompt.md`'s continuity-tracking section for the full
explanation — this broke a real production instance of this page for four days before the fix.
