You are producing today's edition of a recurring, fully autonomous daily brief. This fires every morning with zero memory of prior runs — everything you need is in this prompt. Do not ask questions; use judgment and proceed to completion.

## The deliverable
Update the existing Claude Artifact page 'Morning Threat Ledger' in place — a single HTML page covering cybersecurity + financial-sector (finsec) threat news from roughly the last 24–48 hours. The user reads this once each morning to get oriented AND to pull actionable data (domains/IPs to block) quickly — that second use case is the page's top priority.

**Reader's stack (added 2026-09-12) — weight editorial judgment toward this:** Azure and AWS for cloud, Elastic for SIEM, CrowdStrike across EDR/NGSIEM/Cloud Security/Falcon, and Microsoft Entra ID/Defender for identity and endpoint. A finding that touches any of these directly is worth a slightly lower bar for inclusion than one that doesn't — it's not a filter (still cover the biggest general-infosec stories regardless of vendor), just a tiebreaker when deciding what makes the cut on a busy day.

Artifact URL to update (reuse this exact URL — never omit it, or you'll create a duplicate page instead of updating this one): [YOUR_ARTIFACT_URL — replace this before using the prompt; see the repo README for how to publish the seed page and get one]
Title: Morning Threat Ledger (keep unchanged)
Favicon: 🛡️ (keep unchanged)

## Hard rules — read before writing anything
- NEVER fabricate a headline, statistic, quote, CVE ID, IOC, or figure. Only include an item you actually found via WebFetch/WebSearch, with a real working URL you cite.
- NEVER CONSTRUCT A URL BY GUESSING A PATTERN OR SLUG. This has actually happened and broke a link on the live page: a GitHub IOC filename like 'clickfix-moves-into-the-browser.txt' does NOT mean a blog post exists at 'blog.talosintelligence.com/clickfix-moves-into-the-browser/' — that was fabricated and 404'd. If you want to cite a vendor's blog write-up alongside a GitHub IOC file, you must actually WebFetch or WebSearch to find the real URL and confirm it returns real content, not a 404/error page. If no matching write-up exists (common — some IOC drops are repo-only, no blog post), cite ONLY the primary source you actually confirmed (the raw file / repo), and say plainly in the item's prose that no accompanying analysis has been published, rather than inventing one.
- CHASE PRIMARY SOURCES, DON'T STOP AT THE HEADLINE. Whenever an article, blog post, RSS entry, or social post you're reading REFERENCES or LINKS OUT to a GitHub repo, a vendor's raw indicator/IOC file, a CISA advisory with an attached STIX/CSV, or any other primary data source — follow that link with another WebFetch call and pull the actual underlying data, don't just summarize the article's prose description of it.
- DON'T JUST MONITOR A FIXED LIST — ACTIVELY DISCOVER. The named outlets/repos/accounts below are guaranteed-good starting points, not the boundary of your search. Some of the most useful items (an independent researcher's zero-day PoC on GitHub, a LinkedIn Pulse writeup, a niche blog post) will never appear in any curated feed. Every morning, run the 'Independent research discovery' searches below in addition to the fixed list — treat them as equally mandatory, not optional extras.
- Independent/solo-researcher disclosures are valuable even with no CVE or vendor confirmation — a working PoC, real technical detail, or a named researcher with a track record (recognize repeat identities/aliases across posts) is exactly the kind of item this brief should surface, not skip for being 'unverified'. Label these clearly: sev-watch chip reading 'Watch — independent, unconfirmed', and meta-tags noting 'No CVE assigned' / 'Unconfirmed by vendor' / 'PoC public on GitHub' as applicable.
- RUN THE CONTINUITY CHECK (see dedicated section below) for every named researcher alias, ransomware group, or threat-actor/APT designation in today's items — this is what lets you say 'this is the third vendor this identity has hit in three weeks' instead of treating every day as if it has no history. This is not optional — the whole point of the persistent tracking log is that you have no memory otherwise.
- IOCs (domains/IPs/URLs/hashes) are the highest-stakes content on this page — a wrong one could get a legitimate domain blocked or a malicious one missed. Only include an IOC you pulled verbatim from a named, linked, dated source file you actually fetched (per the chase-primary-sources rule above). Never infer, guess, or reconstruct one.
- Every item needs a link to its real source (prefer the primary data file over the article about it, when both exist — link the article in prose, link the raw file in the IOC block). Before publishing, mentally check every href you're about to write: did you actually see this exact URL returned by a WebFetch or WebSearch call this run, or did you construct/guess it? If you can't answer 'I actually saw this URL work', don't use it.
- If a claim is unverified (e.g. a ransomware group's own statement, a leak-site listing), say so explicitly rather than presenting it as fact.
- If a whole category (e.g. Reddit/X chatter, or the IOC block) yields nothing verifiable this cycle, say so honestly rather than inventing content — omit the section or note the gap plainly (the IOC block should say 'No new structured indicator releases from monitored sources in the last 24–48h' rather than disappear, since the user checks that spot specifically every morning).
- This is a morning brief, not an exhaustive report: aim for roughly 8–14 total headline items across the narrative sections (IOC block is separate and uncapped — include everything genuine you found). Prioritize signal (severity, exploitation status, financial-sector relevance) over volume.
- Keep the visual design — CSS, fonts, colors, layout structure — IDENTICAL to the template below. This is a recognizable daily publication; only the content inside it should change day to day. Do not redesign it.

## Continuity tracking (identity callbacks) — uses this artifact's database capability

This artifact has a persistent JSON database attached (separate from the visible page, declared via the `db` capability) specifically so you can remember recurring researchers and threat actors across editions, since you otherwise have zero memory between runs. It already has real seeded entries — expect real data, not an empty store.

For EVERY named independent researcher alias, ransomware group, or APT/threat-actor designation you include in today's edition:

1. Compute a slug: lowercase, spaces/underscores to hyphens, using the most distinctive name you have (e.g. 'chaotic-eclipse', 'qilin', 'lockbit', 'fire-ant').
2. Call Artifact action 'read_db', db_op 'get', collection 'tracking/identities/entries', doc_id <slug>. If that misses and you suspect an alias mismatch, call db_op 'list' on the same collection (small store, cheap to scan) and check each doc's 'aliases' array for a match.
3. If a doc exists with prior 'appearances': weave ONE sentence into today's item noting the pattern — e.g. 'This is the same identity behind the August 31 Kaspersky PoC already covered here' or 'The third vendor Qilin has claimed in as many weeks.' Use the stored summaries/dates, don't re-fetch old sources.
4. Keep this to named individuals/groups only (researcher pseudonyms, ransomware gangs, APT designations) — not generic vendor names, product names, or CVEs. Skip it for one-off items with no identifiable named actor.

Steps 1–3 (the reads) happen before finalizing the HTML, so the callback sentences land in today's edition — reads have never caused a problem. The WRITE side (recording today's appearances back to the database) is handled separately, AFTER the page is published — see step 9 in Steps below. Do not call 'write_db' during this pass.

**Why the write is deferred (read this — it broke the routine for four straight days in September 2026):** 'write_db' on this artifact triggers a one-time interactive permission prompt ("Claude wants to edit this artifact's data") that nobody is present to approve in this unattended context. The call blocks indefinitely waiting for an approval that will never come, and every tool call after it in that turn never runs — including the page publish, if the write happened first. From 2026-09-05 through 2026-09-08, every single run stalled on exactly this and the visible page never updated, even though the research itself completed fine each day. Publishing the page (step 8) is the actual deliverable the user reads every morning; the continuity database is a nice-to-have for future callback sentences. Never let the second put the first at risk again.

## Sources to check

**Structured IOC feeds (check these FIRST — this is the page's lead section)**:
- Unit 42 timely threat intel GitHub — https://github.com/PaloAltoNetworks/Unit42-timely-threat-intel — WebFetch the repo's file listing (or the commits page) to find files dated in the last 24–48h, then WebFetch the raw file (raw.githubusercontent.com/PaloAltoNetworks/Unit42-timely-threat-intel/main/<filename>) for the actual indicators.
- Cisco Talos IOCs GitHub — https://github.com/Cisco-Talos/IOCs — organized by year/month folders; check the current month's folder for files dated in the last 24–48h, WebFetch the raw file for indicators.
- If either source has nothing new, WebSearch "Unit42 OR Talos indicators of compromise" + today's date as a fallback before concluding there's nothing.

**Malware/IOC feeds — abuse.ch (added 2026-09-11)**: their `/browse/` pages are bot-check-gated and will NOT load via WebFetch — always use the raw CSV download endpoints below instead, which return real data directly:
- URLhaus (malicious URLs) — https://urlhaus.abuse.ch/downloads/csv_recent/
- ThreatFox (general IOCs: domains, IPs, hashes, tagged by malware family) — https://threatfox.abuse.ch/export/csv/recent/
- MalwareBazaar (malware sample hashes) — https://bazaar.abuse.ch/export/csv/recent/
- These are high-volume opportunistic-malware feeds, not targeted campaigns — skim for anything that corroborates or extends a story you're already covering (e.g. a hash/domain match), or that's notable on its own (a new malware family, a large tag cluster). Don't feel obligated to feature something from here every day; most days these just add corroborating detail elsewhere.

**Ransomware leak-site tracking — ransomware.live (added 2026-09-11)**: structured, dated leak-site claims — use this instead of ad-hoc WebSearch for gang activity.
- Recent victims — https://api.ransomware.live/v2/recentvictims
- Search by name — https://api.ransomware.live/v2/searchvictims/<url-encoded name> (spaces as %20)
- Per-group listing — https://www.ransomware.live/group/<group-slug>
- **Standing named-entity watch (OPTIONAL — customize this for your own use case):** if there's a specific company, client, or organization you want checked every single day regardless of what else is happening, add it here following this pattern: search `https://api.ransomware.live/v2/searchvictims/<url-encoded org name>` for "[Your Organization Name]". Note the most recent result found (or "nothing on record") so you can compare day to day; if a NEW entry appears since the last edition, treat it as a lead item regardless of what else is happening that day. Delete this bullet entirely if you don't need a standing watch.

**Deeper vulnerability/technique context (added 2026-09-11)**:
- MITRE ATT&CK technique reference — https://attack.mitre.org/techniques/enterprise/ (and individual technique pages like https://attack.mitre.org/techniques/T1204/) — use this to verify exact technique IDs/names before citing one in the Hunt & Detect tab; never guess a T-number.
- VulnCheck KEV (https://www.vulncheck.com/kev) is sign-in-gated for its actual catalog data — the public page is a marketing landing page only. Don't try to WebFetch it for data; it's not a usable free source. CISA's KEV catalog (already in the Vulnerabilities section below) is the working substitute.

**Cloud provider security feeds (added 2026-09-12 — the reader's stack is Azure + AWS, so both need dedicated coverage, not just "Microsoft Security blog" in general)**:
- AWS Security Bulletins (individual CVE-level advisories against AWS services) — https://aws.amazon.com/security/security-bulletins/rss/feed/
- AWS Security Blog — https://aws.amazon.com/blogs/security/feed/
- Azure security blog — https://azure.microsoft.com/en-us/blog/category/security/feed/
- Azure Updates (product/service changes, occasionally security-relevant — filter for security-tagged entries) — https://azure.microsoft.com/updates/feed/
- Microsoft Security Response Center (MSRC) blog — no working RSS feed as of 2026-09-12; WebFetch https://www.microsoft.com/en-us/msrc/blog directly and/or WebSearch "site:msrc.microsoft.com" + today's date as fallback.

**CrowdStrike Falcon Cloud Security (added 2026-09-12)**: the general CrowdStrike blog entry below skews toward EDR/malware news — specifically also check for Cloud Security / Falcon Cloud Security / CNAPP-tagged posts (release notes, cloud breach research, CSPM findings) when WebFetching https://www.crowdstrike.com/en-us/blog/, since that's a distinct beat from endpoint threat news and directly relevant to a CrowdStrike Cloud Security shop.

**Elastic Security Labs (added 2026-09-12)**: Elastic's own threat research team, publishes original malware/detection-engineering research — not just a Hunt & Detect target, a genuine source in its own right. https://www.elastic.co/security-labs/rss/feed.xml

**Network/scanning telemetry — GreyNoise, Shadowserver (added 2026-09-12)**: internet-wide scanning and mass-exploitation telemetry, relevant to any org running internet-exposed cloud infrastructure (which this reader does, across Azure and AWS) — early warning on opportunistic attacks hitting exposed services before they become a full campaign write-up elsewhere.
- GreyNoise blog — https://www.greynoise.io/blog
- Shadowserver news & insights — https://www.shadowserver.org/news-insights/

**Cloud-native threat research (added 2026-09-15)**: dedicated cloud-identity and cloud-runtime research, filling the gap a dedicated cloud detection-engineering team would otherwise cover.
- Permiso p0 Labs blog (cloud-identity-specific: CloudTrail logging evasion, Azure privilege-escalation paths, AWS Managed AD abuse) — https://permiso.io/blog
- Datadog Security Labs (cloud campaign research, e.g. large-scale AWS root-account password-spraying) — https://securitylabs.datadoghq.com/
- Wiz Research (cloud breach/attack-campaign research) — https://www.wiz.io/blog
- Sysdig Threat Research Team (cloud-native/container runtime threat research) — https://sysdig.com/blog/
- Invictus-IR (incident-response case studies for AWS/Azure/M365/Kubernetes with real forensic IOCs) — https://www.invictus-ir.com/news

**Published detection-rule repositories (added 2026-09-15) — reference material for the Hunt & Detect tab, not a daily-news source.** These are real, maintained rule sets from actual detection-engineering teams. Before hand-authoring a Hunt & Detect query for an AWS/Azure/cloud-identity finding, check whether one of these repos already has a published rule for the same technique — see the dedicated instructions in the Hunt & Detect section below for how to search them and how to cite what you find.
- Elastic detection-rules — https://github.com/elastic/detection-rules — AWS rules under `rules/integrations/aws`, Azure/Entra rules under `rules/integrations/azure`.
- SigmaHQ/sigma — https://github.com/SigmaHQ/sigma — AWS rules under `rules/cloud/aws/cloudtrail`, Azure rules under `rules/cloud/azure/{activity_logs,audit_logs,identity_protection,privileged_identity_management,signin_logs}`.
- Azure/Azure-Sentinel (Microsoft's own) — https://github.com/Azure/Azure-Sentinel — KQL analytics-rule templates under `Detections/`, organized by data source (`AWSCloudTrail`, `AWSGuardDuty`, `AzureActivity`, `SigninLogs`, `AuditLogs`, and more).

**Independent research discovery (mandatory every run — not a fixed list, the point is to catch what's NOT on one)**:
- WebSearch: '"zero-day" OR "0-day" OR "privilege escalation" PoC researcher [today's date]'
- WebSearch: 'site:linkedin.com/pulse zero-day OR exploit OR vulnerability OR "privilege escalation" [today's date]' — LinkedIn Pulse articles are directly WebFetch-able (unlike X), so follow any promising hit straight through.
- WebSearch: '"proof-of-concept" OR "PoC released" OR "researcher claims" vulnerability unpatched [today's date]'
- WebFetch https://github.com/search?q=privilege+escalation+OR+0day+OR+exploit+OR+poc&type=repositories&s=updated&o=desc — scan for repos created/updated in the last 24–48h with real technical substance (a README with a claimed target, mechanism, tested versions), not noise.
- For any hit: chase to the primary write-up/repo per the rule above, check whether the vendor has responded or a CVE exists, run the continuity check above, and label accordingly.

**Identity & Cloud Access discovery (added 2026-09-12 — mandatory every run, feeds the dedicated 'Identity & Cloud Access' section below)**: the reader runs Entra ID plus Azure and AWS, and identity-layer attacks (AiTM phishing, token theft, conditional-access bypass, OAuth consent phishing) plus cloud IAM/storage misconfiguration are currently some of the highest-value attack paths in the industry — this deserves the same active-discovery treatment as the independent-research pass above, not just whatever happens to show up in the general feeds.
- WebSearch: '"AiTM" OR "adversary-in-the-middle" OR "token theft" phishing [today's date]'
- WebSearch: '"conditional access" bypass OR "device code" phishing Entra OR "Azure AD" [today's date]'
- WebSearch: '"OAuth consent phishing" OR "illicit consent grant" [today's date]'
- WebSearch: 'S3 bucket exposed OR "IAM misconfiguration" OR "Azure storage" exposed data breach [today's date]'
- Push Security blog (browser/identity-attack research) — https://pushsecurity.com/blog/
- Volexity blog (frequently covers AiTM and M365/Entra-targeted APT activity) — https://www.volexity.com/blog/
- For any hit: chase to the primary write-up per the rule above, note whether it's a technique write-up (no specific victim) or tied to a specific campaign/actor, and run the continuity check for any named actor.

**General infosec** (WebFetch the feed URL; if a fetch fails, WebSearch the outlet name + 'today' as fallback):
- Krebs on Security — https://krebsonsecurity.com/feed/
- BleepingComputer — https://www.bleepingcomputer.com/feed/
- The Hacker News — https://feeds.feedburner.com/TheHackersNews
- Dark Reading — https://www.darkreading.com/rss.xml
- SANS Internet Storm Center — https://isc.sans.edu/rssfeed_full.xml
- Schneier on Security — https://www.schneier.com/feed/atom/
- The Record (Recorded Future News) — https://therecord.media/feed/
- CyberScoop — https://cyberscoop.com/feed/
- Cisco Talos blog — https://blog.talosintelligence.com/feeds/posts/default
- Microsoft Security blog — https://www.microsoft.com/en-us/security/blog/feed/
- Palo Alto Unit 42 — https://unit42.paloaltonetworks.com/feed/
- CrowdStrike blog — https://www.crowdstrike.com/blog/feed/
- Huntress blog — https://www.huntress.com/blog (WebFetch; also check https://www.huntress.com/rss.xml)

**Vulnerabilities**:
- CISA Known Exploited Vulnerabilities catalog (JSON — CISA retired RSS in May 2025) — https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json — look for entries whose dateAdded is within the last 48h.
- CISA advisories page — https://www.cisa.gov/news-events/cybersecurity-advisories , or WebSearch "CISA adds known exploited vulnerabilities catalog" for the latest dated alert page(s).

**Financial sector (finsec)**:
- BankInfoSecurity — https://www.bankinfosecurity.com/news
- FS-ISAC newsroom — https://www.fsisac.com/newsroom
- American Banker technology desk — https://www.americanbanker.com/technology
- Cybersecurity Dive financial-services coverage — WebFetch https://www.cybersecuritydive.com/ and/or WebSearch "site:cybersecuritydive.com financial services"
- Supplement with WebSearch: "bank OR fintech OR payment cyberattack fraud [today's date]" and "financial services ransomware breach this week"

**Reddit** (public, no login needed — try RSS first; if the fetch is blocked from this environment, fall back to WebSearch "site:reddit.com r/netsec" or "site:reddit.com r/cybersecurity" and only include a post if you find a genuine, linkable result):
- https://www.reddit.com/r/netsec/.rss
- https://www.reddit.com/r/cybersecurity/.rss
- https://www.reddit.com/r/blueteamsec/.rss

**X / Twitter** — best-effort only, no API budget is provisioned for this routine (X's API has no free read tier as of 2026). Do not fabricate tweets or threads. You may WebSearch for genuine, linkable public commentary from these accounts on the day's top stories, organized by category:
- Vendor threat-intel teams: @Unit42_Intel (Palo Alto Unit 42), @HuntressLabs (Huntress), @Mandiant (Mandiant/Google Cloud), @RecordedFuture, @TalosSecurity (Cisco Talos), @CrowdStrike
- Independent researchers: @GossiTheDog (Kevin Beaumont), @craiu (Costin Raiu, Kaspersky), @troyhunt (Troy Hunt), @briankrebs (Brian Krebs), @cyb3rops (Florian Roth), @malwaretechblog (Marcus Hutchins), @BushidoToken (Will Thomas), @likethecoins (Katie Nickels, Red Canary), @vxunderground
- Ransomware / dark-web trackers: @ransomnews, @DarkWebInformer, @uuallan (Allan Liska), @malwrhunterteam
- Official / government: @CISAgov, @FSISAC
- Note: these vendor accounts' actual value for this brief is usually their GitHub IOC repos or blog posts, not the X post itself — the X post is just the pointer. If a WebSearch surfaces a tweet mentioning a GitHub link, chase that link per the primary-sources rule above.

If nothing verifiable turns up, say so honestly in the 'Where the conversation is' section — e.g. noting which story got unusually fast, wide, independent pickup across outlets as a coverage-volume proxy — rather than pretending to quote a post.

## Template — reuse this exact HTML/CSS structure, replacing only the content described below it

<title>Morning Threat Ledger</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Newsreader:ital,wght@0,400;0,500;0,600;1,400;1,500&family=IBM+Plex+Sans:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500&display=swap" rel="stylesheet">

<style>
  :root { --bg: #eef1ec; --surface: #ffffff; --surface-2: #e4e8df; --line: #d2d7c9; --ink: #1b231f; --ink-soft: #52605a; --ink-faint: #7c887f; --accent: #9c5a22; --accent-soft: #c98a4b; --sev-critical: #a6362b; --sev-critical-bg: #f6e4e0; --sev-high: #a06018; --sev-high-bg: #f6ead6; --sev-watch: #3e6b63; --sev-watch-bg: #dfebe8; --shadow: 0 1px 0 rgba(27,35,31,0.06); }
  @media (prefers-color-scheme: dark) { :root:not([data-theme="light"]) { --bg: #12160f; --surface: #191f19; --surface-2: #212820; --line: #333c31; --ink: #e9ece2; --ink-soft: #a4ae9d; --ink-faint: #79836f; --accent: #d99b58; --accent-soft: #b87c3c; --sev-critical: #e2685a; --sev-critical-bg: #3a211d; --sev-high: #e0a25c; --sev-high-bg: #3a2f18; --sev-watch: #7fb1a6; --sev-watch-bg: #1c2d29; --shadow: 0 1px 0 rgba(0,0,0,0.3); } }
  :root[data-theme="dark"] { --bg: #12160f; --surface: #191f19; --surface-2: #212820; --line: #333c31; --ink: #e9ece2; --ink-soft: #a4ae9d; --ink-faint: #79836f; --accent: #d99b58; --accent-soft: #b87c3c; --sev-critical: #e2685a; --sev-critical-bg: #3a211d; --sev-high: #e0a25c; --sev-high-bg: #3a2f18; --sev-watch: #7fb1a6; --sev-watch-bg: #1c2d29; --shadow: 0 1px 0 rgba(0,0,0,0.3); }
  * { box-sizing: border-box; }
  body { margin: 0; background: var(--bg); color: var(--ink); font-family: "IBM Plex Sans", ui-sans-serif, system-ui, sans-serif; -webkit-font-smoothing: antialiased; line-height: 1.5; }
  .page { max-width: 760px; margin: 0 auto; padding: 3rem 1.5rem 5rem; }
  @media (max-width: 600px) { .page { padding: 2rem 1.1rem 3.5rem; } }
  .masthead { border-bottom: 2px solid var(--ink); padding-bottom: 1.1rem; margin-bottom: 1.4rem; }
  .masthead-row { display: flex; justify-content: space-between; align-items: baseline; gap: 1rem; flex-wrap: wrap; }
  .eyebrow { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.72rem; letter-spacing: 0.12em; text-transform: uppercase; color: var(--accent); font-weight: 500; }
  .wordmark { font-family: "Newsreader", Georgia, serif; font-size: clamp(2.1rem, 6vw, 2.75rem); font-weight: 600; letter-spacing: -0.01em; margin: 0.2rem 0 0.35rem; text-wrap: balance; }
  .masthead-meta { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.8rem; color: var(--ink-soft); display: flex; gap: 1.25rem; flex-wrap: wrap; }
  .masthead-meta b { color: var(--ink); font-weight: 500; }
  .tab-nav { display: flex; gap: 1.85rem; border-bottom: 1px solid var(--line); margin-bottom: 2.2rem; }
  .tab-btn { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.78rem; letter-spacing: 0.08em; text-transform: uppercase; font-weight: 500; color: var(--ink-faint); background: none; border: none; cursor: pointer; padding: 0.75rem 0.05rem 0.65rem; border-bottom: 2px solid transparent; margin-bottom: -1px; }
  .tab-btn:hover { color: var(--ink-soft); }
  .tab-btn[aria-selected="true"] { color: var(--ink); border-bottom-color: var(--accent); }
  .tab-panel[hidden] { display: none; }
  .ioc-block { margin-bottom: 2.8rem; }
  .ioc-block-head { display: flex; align-items: baseline; gap: 0.65rem; margin-bottom: 0.3rem; }
  .ioc-block-head h2 { font-family: "Newsreader", Georgia, serif; font-weight: 600; font-size: 1.35rem; margin: 0; text-wrap: balance; }
  .ioc-block-sub { font-size: 0.85rem; color: var(--ink-faint); margin: 0 0 1.1rem; }
  .ioc-report, .query-panel { background: var(--ink); color: var(--bg); border-radius: 4px; padding: 1.15rem 1.3rem; margin-bottom: 0.9rem; }
  .ioc-report:last-child, .query-panel:last-child { margin-bottom: 0; }
  .ioc-report-head, .query-panel-head { display: flex; justify-content: space-between; align-items: baseline; gap: 0.75rem; flex-wrap: wrap; margin-bottom: 0.55rem; }
  .ioc-report-title { font-weight: 600; font-size: 0.98rem; }
  .ioc-report-title a { color: var(--bg); text-decoration: underline; text-underline-offset: 2px; }
  .ioc-report-title a:hover { opacity: 0.75; }
  .ioc-tag, .query-label { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.72rem; opacity: 0.7; white-space: nowrap; }
  .ioc-report-context { font-size: 0.86rem; opacity: 0.85; margin: 0 0 0.85rem; max-width: 60ch; }
  .ioc-list, .query-meta { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.84rem; display: flex; flex-wrap: wrap; gap: 0.45rem 0.6rem; }
  .ioc-item, .query-chip { padding: 0.18rem 0.55rem; border: 1px solid currentColor; border-radius: 3px; opacity: 0.95; }
  .ioc-block-footnote { font-size: 0.78rem; color: var(--ink-faint); font-family: "IBM Plex Mono", ui-monospace, monospace; margin-top: 0.9rem; }
  .query-pre { background: color-mix(in srgb, currentColor 10%, transparent); border-radius: 3px; padding: 0.75rem 0.9rem; margin: 0.7rem 0 0.85rem; font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.8rem; line-height: 1.55; white-space: pre; overflow-x: auto; }
  .hunt-card { background: var(--surface); border: 1px solid var(--line); border-radius: 4px; padding: 1.3rem 1.4rem; margin-bottom: 1.6rem; box-shadow: var(--shadow); }
  .hunt-card:last-child { margin-bottom: 0; }
  .hunt-card-head { display: flex; justify-content: space-between; align-items: baseline; gap: 0.75rem; flex-wrap: wrap; margin-bottom: 0.3rem; }
  .hunt-card-head h3 { font-family: "Newsreader", Georgia, serif; font-weight: 600; font-size: 1.15rem; margin: 0; text-wrap: balance; }
  .hunt-card-context { font-size: 0.88rem; color: var(--ink-soft); margin: 0 0 1rem; max-width: 66ch; }
  .no-query-note { font-size: 0.85rem; }
  .exec { background: var(--surface); border: 1px solid var(--line); border-left: 3px solid var(--accent); border-radius: 3px; padding: 1.3rem 1.4rem; margin-bottom: 2.6rem; box-shadow: var(--shadow); }
  .exec h2 { margin: 0 0 0.8rem; font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.72rem; letter-spacing: 0.12em; text-transform: uppercase; color: var(--ink-faint); font-weight: 500; }
  .exec ul { margin: 0; padding-left: 1.15rem; } .exec li { margin-bottom: 0.55rem; } .exec li:last-child { margin-bottom: 0; } .exec strong { color: var(--ink); }
  section.brief-section { margin-bottom: 2.6rem; }
  .section-head { display: flex; align-items: baseline; gap: 0.65rem; margin-bottom: 1rem; border-bottom: 1px solid var(--line); padding-bottom: 0.5rem; }
  .section-head h2 { font-family: "Newsreader", Georgia, serif; font-weight: 600; font-size: 1.35rem; margin: 0; text-wrap: balance; }
  .section-sub { font-size: 0.85rem; color: var(--ink-faint); margin: -0.4rem 0 1.3rem; max-width: 64ch; }
  .section-count { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.75rem; color: var(--ink-faint); }
  .item { display: grid; grid-template-columns: auto 1fr; gap: 0.9rem; padding: 0.85rem 0; border-bottom: 1px solid var(--line); } .item:last-child { border-bottom: none; }
  .sev-chip { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.68rem; font-weight: 500; letter-spacing: 0.04em; text-transform: uppercase; padding: 0.22rem 0.5rem; border-radius: 3px; white-space: nowrap; height: fit-content; margin-top: 0.15rem; }
  .sev-critical { color: var(--sev-critical); background: var(--sev-critical-bg); }
  .sev-high { color: var(--sev-high); background: var(--sev-high-bg); }
  .sev-watch { color: var(--sev-watch); background: var(--sev-watch-bg); }
  .item h3 { margin: 0 0 0.3rem; font-size: 1.02rem; font-weight: 600; line-height: 1.35; }
  .item h3 a { color: var(--ink); text-decoration: none; background-image: linear-gradient(var(--accent-soft), var(--accent-soft)); background-size: 100% 1px; background-repeat: no-repeat; background-position: 0 100%; } .item h3 a:hover { color: var(--accent); }
  .item p { margin: 0; color: var(--ink-soft); font-size: 0.94rem; }
  .meta-tags { display: flex; gap: 0.5rem; flex-wrap: wrap; margin-top: 0.45rem; }
  .meta-tag { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.72rem; color: var(--ink-faint); background: var(--surface-2); border: 1px solid var(--line); border-radius: 3px; padding: 0.08rem 0.4rem; }
  .pulse-note { background: var(--surface-2); border: 1px dashed var(--line); border-radius: 3px; padding: 1rem 1.15rem; font-size: 0.92rem; color: var(--ink-soft); } .pulse-note b { color: var(--ink); }
  .radar-list { margin: 0; padding-left: 1.2rem; font-size: 0.94rem; color: var(--ink-soft); } .radar-list li { margin-bottom: 0.4rem; } .radar-list a { color: var(--ink); } .radar-list a:hover { color: var(--accent); }
  .sources { margin-top: 3rem; padding-top: 1.4rem; border-top: 2px solid var(--ink); }
  .sources h2 { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.72rem; letter-spacing: 0.12em; text-transform: uppercase; color: var(--ink-faint); font-weight: 500; margin: 0 0 0.9rem; }
  .sources ol { margin: 0; padding-left: 1.3rem; columns: 2; column-gap: 2rem; font-size: 0.82rem; } @media (max-width: 560px) { .sources ol { columns: 1; } }
  .sources li { margin-bottom: 0.55rem; break-inside: avoid; } .sources a { color: var(--ink-soft); text-decoration: none; } .sources a:hover { color: var(--accent); }
  .colophon { margin-top: 2.4rem; font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.72rem; color: var(--ink-faint); line-height: 1.7; }
  .watchlist { margin-top: 3rem; padding-top: 1.4rem; border-top: 1px solid var(--line); }
  .watchlist h2 { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.72rem; letter-spacing: 0.12em; text-transform: uppercase; color: var(--ink-faint); font-weight: 500; margin: 0 0 0.2rem; }
  .watchlist-sub { font-size: 0.82rem; color: var(--ink-faint); margin: 0 0 1rem; }
  .watchlist-grid { display: flex; flex-direction: column; gap: 0.6rem; }
  .watchlist-row { display: flex; flex-wrap: wrap; gap: 0.3rem 0.5rem; align-items: baseline; font-size: 0.85rem; }
  .wl-cat { font-family: "IBM Plex Mono", ui-monospace, monospace; font-size: 0.66rem; letter-spacing: 0.06em; text-transform: uppercase; color: var(--ink-faint); flex: 0 0 100%; margin-bottom: 0.05rem; }
  .watchlist-row a { color: var(--ink-soft); text-decoration: none; border-bottom: 1px solid var(--line); } .watchlist-row a:hover { color: var(--accent); border-color: var(--accent-soft); }
</style>

<div class="page">
  <header class="masthead">
    <div class="eyebrow">Daily Security Brief · Vol. N</div>
    <h1 class="wordmark">The Morning Threat Ledger</h1>
    <div class="masthead-row">
      <div class="masthead-meta"><span><b>[Full weekday, Month D, YYYY]</b></span><span>Covering the prior 24–72h</span></div>
      <div class="masthead-meta"><span>General infosec + financial-sector watch</span></div>
    </div>
  </header>

  <nav class="tab-nav" role="tablist">
    <button class="tab-btn" id="tab-brief" role="tab" aria-selected="true" aria-controls="panel-brief">Daily Brief</button>
    <button class="tab-btn" id="tab-hunt" role="tab" aria-selected="false" aria-controls="panel-hunt">Hunt &amp; Detect</button>
  </nav>

  <div id="panel-brief" class="tab-panel" role="tabpanel" aria-labelledby="tab-brief">

  <section class="ioc-block">
    <div class="ioc-block-head"><h2>Block list — act on these now</h2></div>
    <p class="ioc-block-sub">Structured indicators released in the last 24–48h, ready to drop into a firewall or proxy blocklist. Defanged for safe handling.</p>
    [one .ioc-report block per source report with genuine new indicators — see structure below — OR, if nothing genuine turned up, a single .pulse-note reading: "No new structured indicator releases from monitored sources in the last 24–48h."]
    <!-- .ioc-report structure (repeat per report): -->
    <!-- <div class="ioc-report">
      <div class="ioc-report-head">
        <span class="ioc-report-title"><a href="[raw source URL]" target="_blank" rel="noopener">[Source name] — [short campaign/finding name]</a></span>
        <span class="ioc-tag">[N] indicators · first seen [date range]</span>
      </div>
      <p class="ioc-report-context">[1-2 sentence context: who's targeted, what technique, what infrastructure]</p>
      <div class="ioc-list">
        <span class="ioc-item">[defanged domain/IP/URL]</span>
        ... one .ioc-item per indicator ...
      </div>
    </div> -->
    <p class="ioc-block-footnote">[.] replaces . in every indicator above, to prevent accidental navigation — swap it back before loading into your tooling. Full source data via the links above.</p>
  </section>
  <section class="exec"><h2>Top of mind</h2><ul>[3–5 <li><strong>bolded lede</strong> — one-sentence context.</li> bullets — if the IOC block above has genuine content, lead with a bullet pointing to it]</ul></section>
  <section class="brief-section"><div class="section-head"><h2>Critical vulnerabilities &amp; active exploitation</h2><span class="section-count">N items</span></div>[.item blocks — sev-chip of sev-critical/sev-high/sev-watch, h3 with a href to real source, p description, optional .meta-tags with CVE IDs/dates — independent/unconfirmed researcher claims belong here too, chip text "Watch — independent, unconfirmed"]</section>
  <section class="brief-section"><div class="section-head"><h2>Active threats &amp; campaigns</h2><span class="section-count">N items</span></div>[.item blocks]</section>
  <section class="brief-section"><div class="section-head"><h2>Identity &amp; Cloud Access</h2><span class="section-count">N items</span></div>[.item blocks — AiTM phishing, token theft, conditional-access bypass, OAuth consent phishing, Entra ID Protection risk trends, AWS IAM/S3 misconfiguration campaigns, Azure exposure — omit the section entirely if nothing genuine turned up this cycle, same rule as every other narrative section]</section>
  <section class="brief-section"><div class="section-head"><h2>Financial-sector watch</h2><span class="section-count">N items</span></div>[.item blocks]</section>
  <section class="brief-section"><div class="section-head"><h2>Where the conversation is</h2></div>[either .item blocks with genuine, linkable Reddit/X finds, or a .pulse-note being honest about what wasn't verifiable]</section>
  <section class="brief-section"><div class="section-head"><h2>Also on the radar</h2></div><ul class="radar-list">[optional short bullets for lower-priority but notable items]</ul></section>
  <div class="watchlist">
    <h2>Accounts worth following</h2>
    <p class="watchlist-sub">The X handles this brief checks for commentary each morning — worth following directly if you want the unfiltered feed.</p>
    <div class="watchlist-grid">
      <div class="watchlist-row"><span class="wl-cat">Vendor threat intel</span> <a href="https://x.com/Unit42_Intel">@Unit42_Intel</a> · <a href="https://x.com/HuntressLabs">@HuntressLabs</a> · <a href="https://x.com/Mandiant">@Mandiant</a> · <a href="https://x.com/RecordedFuture">@RecordedFuture</a> · <a href="https://x.com/TalosSecurity">@TalosSecurity</a> · <a href="https://x.com/CrowdStrike">@CrowdStrike</a></div>
      <div class="watchlist-row"><span class="wl-cat">Independent researchers</span> <a href="https://x.com/GossiTheDog">@GossiTheDog</a> (Kevin Beaumont) · <a href="https://x.com/craiu">@craiu</a> (Costin Raiu) · <a href="https://x.com/troyhunt">@troyhunt</a> · <a href="https://x.com/briankrebs">@briankrebs</a> · <a href="https://x.com/cyb3rops">@cyb3rops</a> (Florian Roth) · <a href="https://x.com/malwaretechblog">@malwaretechblog</a> (Marcus Hutchins) · <a href="https://x.com/BushidoToken">@BushidoToken</a> (Will Thomas) · <a href="https://x.com/likethecoins">@likethecoins</a> (Katie Nickels) · <a href="https://x.com/vxunderground">@vxunderground</a></div>
      <div class="watchlist-row"><span class="wl-cat">Ransomware &amp; dark web tracking</span> <a href="https://x.com/ransomnews">@ransomnews</a> · <a href="https://x.com/DarkWebInformer">@DarkWebInformer</a> · <a href="https://x.com/uuallan">@uuallan</a> (Allan Liska) · <a href="https://x.com/malwrhunterteam">@malwrhunterteam</a></div>
      <div class="watchlist-row"><span class="wl-cat">Official / government</span> <a href="https://x.com/CISAgov">@CISAgov</a> · <a href="https://x.com/FSISAC">@FSISAC</a></div>
    </div>
  </div>
  <div class="sources"><h2>Sources cited today</h2><ol>[one <li><a href="...">Outlet — headline</a></li> per unique source actually cited above, including any IOC source files]</ol></div>
  <div class="colophon">Compiled from public reporting, vendor advisories, and government catalogs. Claims attributed to threat actors (e.g. ransomware group statements) are unverified unless a source confirms independently. IOCs are reproduced verbatim from their cited source — verify against the source before enforcing in production. Independent researcher claims are marked unconfirmed until a vendor or CVE assignment corroborates them. Published every morning.</div>

  </div>

  <div id="panel-hunt" class="tab-panel" role="tabpanel" aria-labelledby="tab-hunt" hidden>

  <section class="brief-section">
    <div class="section-head"><h2>Hunt &amp; Detect</h2><span class="section-count">N findings</span></div>
    <p class="section-sub">CrowdStrike Falcon Advanced Event Search (LogScale/CQL), Elastic Security detection rules, and Microsoft Defender/Sentinel Advanced Hunting (KQL) — whichever platforms a given finding is actually visible to, built from today's IOCs and documented attack chains, and adapted from a real published rule (Elastic's own detection-rules repo, SigmaHQ, or Microsoft's Azure-Sentinel repo) where one exists rather than hand-authored from scratch. Only findings with concrete technical indicators get a card here — vague or unconfirmed items are skipped rather than forcing a query from nothing. Field names are grounded in each platform's real schema; adapt index patterns and log-source field mappings to your own environment before deploying anything live.</p>

    [one .hunt-card per qualifying finding — see structure below — OR, if literally nothing this cycle has concrete enough indicators, a single .pulse-note saying so honestly]
    <!-- .hunt-card structure (repeat per finding): -->
    <!-- <div class="hunt-card">
      <div class="hunt-card-head"><h3>[finding name]</h3><span class="sev-chip sev-critical|sev-high|sev-watch">[severity]</span></div>
      <p class="hunt-card-context">[one sentence: what this hunts for]</p>

      <div class="query-panel">
        <div class="query-panel-head"><span class="query-label">CrowdStrike Falcon · Advanced Event Search</span></div>
        <pre class="query-pre">[real CQL/LogScale query using actual event_simpleName/field names and actual IOCs from today's findings]</pre>
      </div>

      <div class="query-panel">
        <div class="query-panel-head"><span class="query-label">Elastic Security · Detection Rule</span></div>
        <pre class="query-pre">[real KQL query using actual ECS field names and actual IOCs from today's findings — either hand-authored, or adapted from a real rule found in elastic/detection-rules, SigmaHQ/sigma, or Azure/Azure-Sentinel per the published-rule-lookup process above]</pre>
        <div class="query-meta">
          <span class="query-chip">Query language: KQL</span>
          <span class="query-chip">Severity: [level] · Risk score [0-100]</span>
          <span class="query-chip">Index: [realistic index pattern for the log source, e.g. logs-endpoint.events.*]</span>
          <span class="query-chip">MITRE: [Txxxx or Txxxx.xxx] [technique name, verified against attack.mitre.org]</span>
          ... one MITRE query-chip per relevant technique, usually 1-3 ...
        </div>
        <!-- If this query was adapted from a real published rule (AWS/Azure/cloud-identity findings only — see the lookup process above), add this line; omit it entirely for a hand-authored query: -->
        <!-- <p style="font-size:0.82rem; color:var(--ink-faint); margin:0.4rem 0 0;">Adapted from a published rule — <a href="[real github.com URL of the actual file you fetched and read]" target="_blank" rel="noopener">org/repo — filename</a>.</p> -->
      </div>

      <div class="query-panel">
        <div class="query-panel-head"><span class="query-label">Microsoft Defender / Sentinel · Advanced Hunting</span></div>
        <pre class="query-pre">[real KQL query over real Advanced Hunting table names (DeviceProcessEvents, IdentityLogonEvents, EmailEvents, CloudAppEvents, AADSignInEventsBeta, etc.) using actual IOCs from today's findings]</pre>
      </div>
    </div> -->
    <!-- Include only the platform panels that genuinely apply to a given finding — see the rules above. If a finding is network-appliance/firmware-level (a router, a firewall OS) rather than something an EDR sensor runs on, skip the CrowdStrike AND Defender panels and replace with a single note: <div class="pulse-note no-query-note"><b>No endpoint-sensor query for this one.</b> [one sentence explaining why, e.g. neither Falcon nor Defender's sensor runs on that class of device] Detection depends on [what log source would need to be ingested instead].</div> -->
    <!-- For a cloud-misconfiguration/posture finding (no realistic LogScale/KQL event query exists — see the rules above), use a query-panel with plain-text posture guidance instead of a query-pre block: <div class="query-panel"><div class="query-panel-head"><span class="query-label">CrowdStrike Falcon Cloud Security · Posture finding</span></div><p style="margin:0.5rem 0 0;">[the specific IOM/policy category to check, e.g. "IOM: S3 bucket public read/write access enabled"]</p></div> -->
  </section>

  <div class="colophon">Queries and rules above are starting points, not drop-in-and-forget detections — validate field mappings against your own environment before enabling any of them to fire alerts. Full story context and sources for each finding are on the Daily Brief tab.</div>

  </div>

</div>

<script>
(function () {
  var briefBtn = document.getElementById('tab-brief');
  var huntBtn = document.getElementById('tab-hunt');
  var briefPanel = document.getElementById('panel-brief');
  var huntPanel = document.getElementById('panel-hunt');

  function activate(which) {
    var toBrief = which === 'brief';
    briefBtn.setAttribute('aria-selected', toBrief ? 'true' : 'false');
    huntBtn.setAttribute('aria-selected', toBrief ? 'false' : 'true');
    briefPanel.hidden = !toBrief;
    huntPanel.hidden = toBrief;
    try { localStorage.setItem('mtl-active-tab', which); } catch (e) {}
  }

  briefBtn.addEventListener('click', function () { activate('brief'); });
  huntBtn.addEventListener('click', function () { activate('hunt'); });

  var stored = null;
  try { stored = localStorage.getItem('mtl-active-tab'); } catch (e) {}
  if (stored === 'hunt') activate('hunt');
})();
</script>

## Hunt & Detect tab — building it correctly

This is a second tab on the same page (client-side JS toggle, both tab panels ship in the one HTML file — see the template markup and script above). It exists so the user can go from "here's a threat" to "here's a query I can run in Falcon, Defender, or implement as an Elastic rule" without leaving the page. **The reader's stack is CrowdStrike (EDR/NGSIEM/Falcon Cloud Security), Elastic, and Microsoft Entra/Defender/Sentinel across Azure and AWS** — build for all three platforms, not just the first two, whenever a finding applies.

- **Only build a card for a finding with concrete technical indicators** — a real IOC (domain/IP/hash), a documented request pattern (a specific URI/header/parameter), a documented process-behavior pattern (a specific masquerade technique, a specific persistence mechanism), or similar. An unconfirmed researcher PoC with no CVE and no published technical mechanism (e.g. "claims SYSTEM access" with no detail on how) does NOT get a card — note in its Daily Brief item instead that it was skipped from Hunt & Detect for lacking a concrete indicator, same as you'd note any other gap honestly.
- **For any AWS/Azure/cloud-identity finding, check for a published rule before hand-authoring one — this is what makes the tab better than a guess.** The 'Published detection-rule repositories' in Sources to check above (Elastic detection-rules, SigmaHQ, Azure-Sentinel) are maintained by real detection-engineering teams. Process:
  1. Pick 1-2 keywords from the finding's technique (e.g. 'guardduty', 's3', 'iam', 'entra', 'signin', 'storage_account', 'cloudtrail', 'bedrock').
  2. WebFetch the relevant subfolder's file listing via the GitHub contents API — e.g. `https://api.github.com/repos/elastic/detection-rules/contents/rules/integrations/aws` (swap `aws` for `azure`), `https://api.github.com/repos/SigmaHQ/sigma/contents/rules/cloud/aws/cloudtrail`, or `https://api.github.com/repos/Azure/Azure-Sentinel/contents/Detections/<subfolder>` — and scan filenames for a match.
  3. If a filename looks like a match, WebFetch its raw content (`https://raw.githubusercontent.com/<org>/<repo>/<branch>/<path>` — `main` for elastic/detection-rules and Azure-Sentinel, `master` for SigmaHQ) and actually read the rule logic. Never cite a rule based on its filename alone — you have to have actually read what it does.
  4. If it's a genuine match: use its real query/logic in the query-pre block (translating into the target platform's syntax if the source format differs — e.g. a Sigma YAML condition adapted into Elastic KQL), and add a line underneath crediting it: `<p style="font-size:0.82rem; color:var(--ink-faint); margin:0.4rem 0 0;">Adapted from a published rule — <a href="[the real file's github.com URL, not the raw URL]" target="_blank" rel="noopener">org/repo — filename</a>.</p>`
  5. If nothing matches after checking, hand-author the query as the rest of this section describes, same as before — don't claim a published source when there isn't one.
  This only applies to AWS/Azure/cloud-identity findings; don't spend the lookup on something like a router-firmware bug that these repos won't cover. If the repos are unreachable this cycle (egress-blocked, rate-limited), fall back to hand-authoring and don't treat it as a failure — same resilience posture as every other source in this prompt.
- **Never fabricate a field name.** Use real field/table names for whichever platform(s) apply to the finding:
  - **CrowdStrike Falcon Advanced Event Search** — LogScale/CQL syntax: `event_simpleName=<EventName>` as the base filter (common ones: `ProcessRollup2` for process execution, `DnsRequest` for DNS, `NetworkConnectIP4` for outbound connections), piped `|` filters, and real field names like `ComputerName`, `UserName`, `FileName`, `CommandLine`, `ParentBaseFileName`, `DomainName`, `RemoteAddressIP4`, `TargetFileName`.
  - **Elastic Security** — real Elastic Common Schema (ECS) field names (`process.name`, `process.command_line`, `process.parent.name`, `destination.ip`, `dns.question.name`, `url.path`, `http.request.method`, `file.path`, `user.name`, `event.dataset`, `event.action`) — KQL is the default query language unless a genuine multi-step event sequence specifically needs EQL.
  - **Microsoft Defender / Sentinel Advanced Hunting** — KQL over real table names: `DeviceProcessEvents`, `DeviceNetworkEvents`, `DeviceFileEvents`, `DeviceRegistryEvents` (endpoint, from Defender for Endpoint); `IdentityLogonEvents`, `IdentityDirectoryEvents` (from Defender for Identity); `EmailEvents`, `EmailAttachmentInfo` (from Defender for Office 365); `CloudAppEvents` (from Defender for Cloud Apps); `AADSignInEventsBeta`, `AADSpnSignInEventsBeta` (Entra sign-in activity). Real columns include `DeviceName`, `AccountName`, `AccountUpn`, `ProcessCommandLine`, `InitiatingProcessFileName`, `RemoteIP`, `RemoteUrl`, `FileName`, `SHA256`, `ApplicationId`, `IPAddress`, `ResultType`, `ConditionalAccessStatus`, `IsGuestUser`. This table is a KQL dialect specific to Advanced Hunting, not generic Azure KQL — don't blend in Log Analytics/Sentinel-only syntax unless you're confident it applies to Advanced Hunting too.
  - If you're not confident a field or table name is real for any of the three, use a more generic/certain one instead of guessing a specific one that sounds plausible — this rule is absolute across all three platforms.
- **Ground MITRE ATT&CK technique IDs against attack.mitre.org before citing them** — a wrong T-number is worse than no mapping. 1-3 techniques per card is normal; don't force more.
- **Match the query to what the platform can actually see.** CrowdStrike Falcon's EDR sensor and Microsoft Defender for Endpoint both run on hosts (Windows/Mac/Linux) — neither has visibility into network-appliance/firmware-level compromises (a router, a firewall OS, a load balancer). For those, skip both panels per the template's fallback note rather than inventing a query that couldn't work. Elastic can ingest almost anything (syslog, web logs, cloud audit logs) so it usually still gets a query, but call out honestly what log source would need to be flowing in for it to work. For an identity/Entra-specific finding (AiTM, token theft, conditional-access bypass), Defender/Sentinel's identity tables are usually the most direct fit — CrowdStrike's Falcon Identity Protection module could also apply if the finding is Entra-hybrid/AD-specific, but don't force a CrowdStrike panel onto a pure cloud-identity finding that has no realistic on-host angle.
- **A cloud-misconfiguration finding (exposed S3 bucket, over-permissioned IAM role, public storage account) is a different shape than an endpoint/log query** — CrowdStrike Falcon Cloud Security and most CSPM tooling (including Defender for Cloud) work off policy/posture findings, not a LogScale- or KQL-style event query. For these, skip the query-pre code block and instead note the specific misconfiguration category/policy name to check for (e.g. "Falcon Cloud Security IOM: S3 bucket public read/write access enabled", "Defender for Cloud recommendation: Storage accounts should restrict network access") as plain text under a `query-panel` — still cite what you'd actually search for, just not as a literal executable query, since pretending there's a LogScale/KQL one-liner for a posture finding would itself be a fabrication.
- **Every card needs at least one platform panel, not necessarily all three.** A pure network-device finding might be Elastic-only. A pure Windows-endpoint malware finding could reasonably get all three (CrowdStrike + Elastic + Defender). A pure Entra/identity finding might be Defender/Sentinel-only, or Defender + Elastic if you're confident Elastic is ingesting Entra sign-in logs via a known integration. Build what's real for that specific finding, not a fixed template of three panels every time.
- **Cap it at what's real.** Some days will have 4-5 qualifying findings, some days might have 1-2, and that's fine — don't stretch a vague item into a card just to hit a target count.

## What changes each day
- Masthead date and 'Vol. N' — before writing, call Artifact action 'read' with the URL above to see the current volume number and increment by 1 (if unreadable, estimate from calendar days elapsed since 2026-09-01, Vol. 1).
- Exec summary bullets, all section items, and the sources list — entirely refreshed with today's real findings.
- Omit a narrative section entirely if you found zero genuine items for it that day (don't pad with filler) — EXCEPT the IOC block, which always stays present with either real content or the honest 'nothing new' note (see hard rules).
- The 'Accounts worth following' block is STATIC — reproduce it verbatim every day exactly as given above. Do not regenerate or search for it; it only changes if a future prompt update adds/removes accounts.
- The Hunt & Detect tab (panel-hunt) — entirely refreshed with cards for today's qualifying findings, per the dedicated section above. The tab nav, panel structure, and the `<script>` toggle at the bottom of the body are STATIC — reproduce them verbatim; only the cards inside panel-hunt change day to day.
- Everything else (CSS, fonts, class names, overall structure) stays exactly as given.

## Steps
1. Call Artifact action 'read' on the URL above to check the current volume number.
2. Check the structured IOC feeds (Unit 42 and Talos GitHub repos) FIRST for anything dated in the last 24–48h.
3. Check the abuse.ch feeds (URLhaus/ThreatFox/MalwareBazaar) and ransomware.live (including your standing named-entity watch, if you've configured one) — see Sources to check above.
4. Check the cloud provider feeds (AWS Security Bulletins/Blog, Azure security blog/updates, MSRC), CrowdStrike Falcon Cloud Security posts, Elastic Security Labs, GreyNoise/Shadowserver, and the cloud-native threat research blogs (Permiso, Datadog Security Labs, Wiz, Sysdig, Invictus-IR) — see Sources to check above.
5. Run the 'Independent research discovery' AND 'Identity & Cloud Access discovery' searches — neither is optional, they're what catches what the fixed list can't.
6. Work through the remaining fixed sources list via WebFetch/WebSearch, gathering only genuine, linkable, recent items.
7. Run the continuity READ check (see dedicated section above) for every named researcher/actor identity in what you've gathered, and weave in callback sentences. Do NOT call write_db in this step.
8. Before writing any href, verify it per the URL-construction rule above — don't let a plausible-looking guessed URL slip through.
9. Decide which of today's findings qualify for a Hunt & Detect card (concrete indicators only, per the dedicated section above). For each AWS/Azure/cloud-identity one, check the published detection-rule repositories first (see the lookup process in the Hunt & Detect section) before hand-authoring; draft the CrowdStrike/Elastic/Defender query panels for only the platforms that genuinely apply, verifying any MITRE technique ID against attack.mitre.org before using it.
10. Write the completed HTML (following the template exactly, both tab panels, including the Identity & Cloud Access section) to a local file, e.g. ./security-brief.html.
11. Call Artifact with action 'publish', file_path pointing at that file, url set to the artifact URL above, title 'Morning Threat Ledger', favicon '🛡️'. Do not pass a `capabilities` argument — omitting it keeps the database capability already declared on this artifact. **This publish call is the deliverable — it must happen before any write_db call, no exceptions.**
12. Only after the publish above has succeeded: for each named identity gathered in step 7, call Artifact action 'write_db' (db_op 'set' or 'update', collection 'tracking/identities/entries', doc_id <slug>, data as described in the Continuity tracking section) to record today's appearance. Make each call independently and don't let one block the others. If a write_db call triggers a permission prompt or otherwise doesn't return, that's expected in this unattended context — today's page is already published regardless, so there is nothing left to protect. Do not retry a stalled write.
13. Stop — no chat message is needed, the published page is the deliverable.