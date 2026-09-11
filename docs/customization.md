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

## Swapping the Hunt & Detect platforms

The tab is built around CrowdStrike Falcon (Advanced Event Search / LogScale query language) and
Elastic Security (KQL detection rules) because that's what the original deployment runs. If you use
different tooling — Microsoft Defender for Endpoint (KQL over Advanced Hunting tables like
`DeviceProcessEvents`), Splunk (SPL), Sentinel (KQL over different table names), Chronicle/Google
SecOps (YARA-L) — edit the `## Hunt & Detect tab — building it correctly` section:

- Replace the CrowdStrike field-name list with your platform's real field/table names.
- Replace `query-label` text in the template ("CrowdStrike Falcon · Advanced Event Search") with
  your platform's name.
- Keep the core rule intact: **only build a card when there's a concrete indicator to hunt on**,
  and **never fabricate a field or table name** — if you're not sure a field exists, say so or use
  a more generic one you're confident about, the same discipline that's already in the prompt for
  CrowdStrike/Elastic. A wrong field name in a query someone might actually deploy is a worse
  failure than an honest gap.

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
