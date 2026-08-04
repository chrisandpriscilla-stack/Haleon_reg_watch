# Green Heat Incentives Agent — run book

This file is the source of truth for the recurring agent that finds public funding
for supplier heat decarbonisation. The scheduled Routine's prompt is generated from
the "Agent prompt" section below — edit here first, then update the Routine so the
two stay in step.

## Purpose

Haleon's Green Heat programme removes two barriers for suppliers burning fossil fuel
for process heat: capital (Johnson Controls heat-as-a-service, no upfront spend) and
commercial pull (carbon pricing in tenders). Public grants are the third lever, and
the one nobody is systematically tracking. Most of this funding is **local** — county,
combined-authority and devolved-nation schemes with small windows and low awareness —
so it only becomes usable if someone maps it to the specific counties where suppliers
actually run heat assets.

The agent's job is **detection and triage**, not application writing. Output feeds the
supplier-specific "value-at-stake" one-pager that category managers take into QBRs and
tenders.

## Inputs

`supplier-sites.csv` in this directory drives the geography. One row per supplier
**site** (not per supplier — a supplier with three plants gets three rows, because
incentives are site-located). Columns:

| column | meaning |
| --- | --- |
| `site_id` | stable key, your choice of format |
| `supplier_name` | legal or trading name |
| `site_name` | plant identifier |
| `country` | e.g. United Kingdom, Spain, Poland |
| `region_or_county` | **the targeting field** — county (UK), province/autonomous community (ES), voivodeship (PL) |
| `postcode_or_zip` | narrows to combined-authority / LEP-successor schemes |
| `current_heat_source` | e.g. natural gas steam boiler, LPG, heavy fuel oil |
| `annual_heat_demand_mwh` | sizes the opportunity; leave blank if unknown |
| `annual_heat_tco2e` | Scope 1 for the supplier, Scope 3 Cat. 1 for Haleon |
| `haleon_category` | procurement category |
| `haleon_annual_spend_gbp` | prioritisation weight |
| `category_manager` | who owns the relationship and carries the message |
| `notes` | anything else |

See `supplier-sites.example.csv` for the shape. Only `country` and `region_or_county`
are strictly required for the agent to work; everything else improves prioritisation.

**If `supplier-sites.csv` has no data rows**, the agent runs in *jurisdiction-default
mode*: it covers the national and devolved layer for the UK only, and states plainly at
the top of the digest that county targeting is disabled pending the site list. It must
not invent supplier names, sites or locations.

## Scope — what counts as a green heat incentive

In scope:

- Capital grants for fuel switching away from gas/oil/LPG process heat — heat pumps
  (including high-temperature and waste-heat recovery), electric boilers, electrode
  boilers, biomass and biomethane where locally permitted, solar thermal
- Heat network connection funding where a supplier site can offtake
- Energy efficiency grants that materially cut heat demand (recovery, insulation,
  controls) where they are a precursor to fuel switching
- Feasibility-study and technical-audit funding, including free diagnostic visits —
  these are the cheapest first step and the natural pairing with a JCI desktop audit
- Soft loans, subsidised finance and tax reliefs where the subsidy element is real
- Devolved, combined-authority, county, district and city schemes — **this layer is the
  point of the agent**, not the national headline funds

Out of scope: domestic/household schemes, EV and transport funding, pure renewable
*electricity* funding (that is the Schneider Energize track — note it and move on),
R&D-only calls with no deployment pathway, and anything a supplier cannot apply for
directly.

## Sources to check each run

National and devolved layer, checked every run:

- GOV.UK funding and support for business finder, filtered to energy/decarbonisation
- DESNZ scheme pages — Green Heat Network Fund, Heat Network Efficiency Scheme, and any
  successor to the Industrial Energy Transformation Fund (IETF closed to new
  applications; watch for the announced-but-unconfirmed successor)
- gov.scot / Scottish Enterprise — Scottish IETF, Scotland's Heat Network Fund
- Business Wales and Welsh Government energy schemes
- Invest NI / NI Department for the Economy
- UK Infrastructure Bank / National Wealth Fund where relevant to heat networks

County and regional layer, checked for **each distinct `region_or_county` in the site
list**:

- The county or unitary council's business support and climate pages
- The relevant combined authority or mayoral authority
- The local Growth Hub or its post-2026 successor body
- Any Net Zero Hub covering that geography

Non-UK sites: the national programme plus the regional government of the named
province or autonomous community, plus EU-level instruments where a supplier could
realistically apply (Innovation Fund, Modernisation Fund, LIFE, national
Recovery-and-Resilience allocations).

Always search with the current month and year in the query — grant windows turn over
fast and cached summaries go stale within weeks.

### Verification constraint

As at the baseline run, the execution environment's network policy blocked direct
fetches of `www.gov.uk`, `assets.publishing.service.gov.uk`, `www.gov.scot` and
`businesswales.gov.wales` (CONNECT refused with 403), so scheme details could only be
gathered through web search rather than read off the administering body's own page.
Retry the direct fetch each run — the policy may change — and where it still fails, mark
the affected figures **unconfirmed** rather than dropping them. A blocked fetch is worth
one line in the digest so the reader knows why confidence is qualified; it is not a
reason to skip the source.

## State handling

Read the previous snapshots from `sources/green-heat/`. On the first run these will be
absent — skip the diff and treat everything found as new, saying so at the top.
Thereafter, compare and surface **changes**: newly opened windows, closing dates now
inside 90 days, schemes newly closed or exhausted, eligibility or intensity changes,
and new schemes in a county that previously had none. Do not re-list an unchanged
scheme in full; a single line confirming it is still open is enough.

## Output — `incentives/<YYYY-MM-DD>.md`

UK English, written for a senior procurement audience who will act on it, not read it
for interest. Structure:

1. **Header** — run date, how many sites and counties were covered, and whether county
   targeting was active or defaulted.
2. **Closing soon** — anything with a deadline inside 90 days, most urgent first. This
   section leads because it is the only part with a clock on it.
3. **By county** — one block per county that has at least one live scheme. Name the
   suppliers and sites in that county from the site list so the category manager can
   see immediately who it applies to.
4. **National and devolved** — the layer that applies regardless of county.
5. **Changes since last run** — opened, closed, changed.
6. **No-coverage counties** — counties in the site list where nothing live was found.
   Say so explicitly; silence reads as an oversight.

For each scheme give: name and administering body; what it funds; grant size and
intensity (% of project cost); who is eligible, including SME-only restrictions and
whether a supplier to a large corporate qualifies; the deadline or "rolling"; the
application link; and one line on fit — which listed sites could plausibly use it and
what the next step is.

Flag confidence honestly. Where a scheme is sourced only from an intermediary or
consultancy page rather than the administering body, mark it **unconfirmed** and say so.
Never state a deadline or a grant ceiling you have not seen on an official page.

End with one line: **Recommend action** — the single highest-value follow-up this week.

If nothing material changed, the body is exactly:
`No material changes to green heat incentives this week.` Do not pad.

## Cadence

Monthly is the right rhythm for this — grant windows move slower than regulation, and a
weekly run would mostly emit the no-change line. The exception is a quarter-end or
programme-end crunch, when windows close in clusters; raise the cadence to fortnightly
if the "closing soon" section is regularly non-empty.

## Delivery

Git archive only. No email, no external sends. Chris reads `incentives/<date>.md` on
GitHub.

## Agent prompt

The text below is what the scheduled Routine runs. Keep it self-contained — it fires
into a fresh session with no memory of this repo.

---

You are a public-funding scout for a Haleon procurement sustainability lead. Your job is
DETECTION AND TRIAGE of government grants that help Haleon's suppliers switch process
heat off fossil fuels. You do not write applications.

=== REPO CONTEXT ===
The repo chrisandpriscilla-stack/Haleon_reg_watch is cloned for you and your working
directory IS the repo root. Run `pwd && ls -la` first to confirm. All paths below are
relative to that root — no leading slashes. Read
`agents/green-heat-incentives/AGENT.md` in full before doing anything else; it is the
run book and it governs scope, sources, output format and confidence rules. Follow it.

=== INPUT ===
Read `agents/green-heat-incentives/supplier-sites.csv`. The `country` and
`region_or_county` columns define which jurisdictions you search. If the file has only a
header row, run in jurisdiction-default mode as the run book describes and say so at the
top of the digest. Never invent supplier names, sites or locations.

=== WORK ===
Check the national/devolved sources listed in the run book every time, then the county
and regional layer for each distinct region_or_county in the site list. Search with the
current month and year in your queries. Read previous snapshots from
`sources/green-heat/` and diff against them; on the first run there will be none, so
treat everything as new.

=== OUTPUT ===
Write `incentives/<YYYY-MM-DD>.md` using the six-section structure in the run book.
Lead with anything closing inside 90 days. Mark as **unconfirmed** any scheme you could
only source from a consultancy or aggregator page rather than the administering body,
and never state a deadline or grant ceiling you have not seen on an official page. End
with a single `Recommend action` line.

=== SNAPSHOTS ===
Write one file per source under `sources/green-heat/` — use `national-uk.md`,
`devolved-<nation>.md` and `county-<slug>.md` — holding this run's fetched content, so
the next run can diff against it.

=== ARCHIVE (GIT) — DO ALL STEPS IN ORDER ===
Substitute <date> with today in YYYY-MM-DD format.
1. `pwd && ls -la` — confirm repo root.
2. `mkdir -p incentives sources/green-heat`
3. Write the digest and the snapshots with the Write tool.
4. `git status` — verify changes present.
5. `git add incentives/ sources/green-heat/`
6. `git -c user.email=agent@anthropic.com -c user.name="Green Heat Incentives Agent" commit -m "Green heat incentives <date>"` — if the commit fails because nothing changed, log it and continue.
7. PUSH EXPLICITLY: `git push origin HEAD:claude/green-heat-incentives-agent-ub25c5`. If rejected as non-fast-forward, run `git push --force-with-lease origin HEAD:claude/green-heat-incentives-agent-ub25c5`. If push fails for auth or permission reasons, capture the exact error text.
8. `git ls-remote origin claude/green-heat-incentives-agent-ub25c5` — VERIFY the remote branch shows your new SHA. The archive only counts as OK if it does.

=== FINAL STATUS LINE ===
The very last line of your output MUST be exactly one of:
- `ARCHIVE OK sha=<short-sha> files=<count> pushed=yes`
- `ARCHIVE SKIP no-changes`
- `ARCHIVE FAIL <one-line reason including any git push error text>`
Do not omit it.
