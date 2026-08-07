# Green Heat Incentives Agent — run book

This file is the source of truth for the weekly agent that finds public financial support
for supplier heat decarbonisation. The scheduled Routine's prompt is generated from the
"Agent prompt" section at the end — edit here first, then update the Routine so the two
stay in step.

## Purpose

Help Haleon's suppliers decarbonise fossil fuel usage and transition to low-carbon
heating. Identify what government and state incentives, grants, subsidies and other
financial support are available; what each one incentivises; what it can be used for;
and how to apply. Present it as a table by country, refreshed weekly.

This sits alongside the two levers the Green Heat programme already has — Johnson
Controls heat-as-a-service removing the capex barrier, and carbon pricing in tenders
creating commercial pull. Public money is the third. Output feeds the supplier-specific
"value-at-stake" one-pager that category managers take into QBRs and tenders.

The agent does **detection and triage**. It does not write applications.

## Inputs

`supplier-sites.csv` in this directory drives which countries are covered. One row per
supplier **site**, not per supplier — a supplier with plants in three countries generates
three rows, because incentives are located where the asset is.

| column | meaning |
| --- | --- |
| `site_id` | stable key, your choice of format |
| `supplier_name` | legal or trading name |
| `site_name` | plant identifier |
| `country` | **the primary targeting field** — drives which national programmes are searched |
| `region_or_county` | secondary targeting — US state, German Land, Spanish autonomous community, UK county |
| `postcode_or_zip` | narrows to local schemes |
| `current_heat_source` | e.g. natural gas steam boiler, LPG, heavy fuel oil |
| `annual_heat_demand_mwh` | sizes the opportunity; blank if unknown |
| `annual_heat_tco2e` | Scope 1 for the supplier, Scope 3 Cat. 1 for Haleon |
| `haleon_category` | procurement category |
| `haleon_annual_spend_gbp` | prioritisation weight |
| `category_manager` | who owns the relationship and carries the message |
| `notes` | anything else |

See `supplier-sites.example.csv` for the shape. Only `country` is strictly required;
`region_or_county` unlocks the sub-national layer, and the rest improves prioritisation.

**If `supplier-sites.csv` has no data rows**, the agent runs in *default-coverage mode*:
it covers the standing country set below and states plainly at the top of the digest that
the country list is a default, not Haleon's actual supplier footprint. It must never
invent supplier names, sites or locations.

**Standing country set** (used until the site list is populated, and always worth keeping
in view because supplier footprints shift): United Kingdom, Ireland, Germany, France,
Netherlands, Spain, Italy, Poland, United States, India, China — plus the EU-wide layer,
which applies across all EEA countries and is often the largest single instrument
available.

## Scope — what counts

In scope:

- Capital grants and subsidies for fuel switching off gas, oil, LPG and coal process heat
  — heat pumps including high-temperature, electric and electrode boilers, biomass and
  biomethane where locally permitted, solar thermal, waste-heat recovery
- Operating subsidies and contracts-for-difference that close the running-cost gap
  (SDE++, Klimaschutzverträge, and similar)
- Competitive auctions for industrial heat decarbonisation
- Tax instruments with a real subsidy element — accelerated depreciation, investment
  allowances, tax credits
- Soft loans and subsidised finance
- Heat network connection funding where a supplier site can offtake
- Feasibility studies, energy audits and free diagnostic visits — the cheapest first step
  and the natural pairing with a JCI desktop audit
- Sub-national schemes: US states, German Länder, Spanish autonomous communities, UK
  counties and combined authorities, Indian states

Out of scope: domestic and household schemes, transport and EV funding, pure renewable
*electricity* funding (that is the Schneider Energize track — note and move on), R&D-only
calls with no deployment pathway, and anything a supplier cannot apply for directly or as
a named partner.

## Required fields for every scheme

The instruction is explicit that each entry must answer all of these. An entry missing
any of them is incomplete:

1. **Country** and, where relevant, sub-national geography
2. **Scheme name and administering body**
3. **What it incentivises** — the behaviour or outcome being paid for
4. **What it can be used for** — eligible costs and technologies, concretely
5. **Value** — grant ceiling, intensity as a % of project cost, or subsidy rate, plus any
   SME uplift
6. **Eligibility** — who can apply, including SME-only restrictions and whether a
   supplier to a large corporate qualifies
7. **How to apply** — the mechanism: rolling application, competitive call, auction,
   tax return, or via an installer or accredited partner; and the practical first step
8. **Deadline** — a date, "rolling", or the expected opening of the next window
9. **Link** to the administering body
10. **Confidence** — see below

## Confidence discipline

Never state a deadline, grant ceiling or intensity you have not seen on an official page.
Where a scheme is sourced only from a consultancy, aggregator or news page, mark it
**unconfirmed** and say so in the row. Suppliers make investment decisions on this; a
wrong deadline is worse than an absent one.

### Verification constraint

As at the baseline run, the execution environment's network policy blocked direct
fetches of `www.gov.uk`, `assets.publishing.service.gov.uk`, `www.gov.scot` and
`businesswales.gov.wales` (CONNECT refused with 403), so scheme details could only be
gathered through web search rather than read off the administering body's own page.
Retry the direct fetch each run — the policy may change — and where it still fails, mark
the affected figures **unconfirmed** rather than dropping them. Note the blocked domains
in one line in the digest so the reader knows why confidence is qualified. A blocked
fetch is not a reason to skip a source.

## Sources to check each run

**EU-wide layer** — check every run regardless of country mix, because it is frequently
the largest instrument available:

- Innovation Fund, especially the Heat Auction series (DG CLIMA and CINEA)
- Modernisation Fund, LIFE programme
- EIB and the EU industrial decarbonisation financing mechanisms
- National Recovery and Resilience Plan allocations still running

**Per country in the site list** (or the standing set), the national energy or climate
agency and its live calls — for example DESNZ and gov.uk (UK), SEAI (Ireland), BMWE/KEI
(Germany), ADEME (France), RVO (Netherlands), IDAE (Spain), GSE and MASE (Italy), NFOŚiGW
(Poland), DOE and IRS (United States), BEE (India), NDRC (China).

**Sub-national layer** for each distinct `region_or_county`: the state, Land, region or
county authority, plus any local growth or net-zero body.

Always include the current month and year in search queries — windows turn over fast and
cached summaries go stale within weeks.

## State handling

Read previous snapshots from `sources/green-heat/`. On the first run these are absent —
skip the diff and say so. Thereafter surface **changes**: newly opened windows, deadlines
now inside 90 days, schemes closed or exhausted, changes to value or eligibility, and new
schemes in a country that previously had none. Do not re-list an unchanged scheme in
full; one line confirming it is still open is enough. The full country tables are
regenerated each run so the digest stands alone, but the narrative sections lead with
what moved.

## Output — `incentives/<YYYY-MM-DD>.md`

UK English, for a senior procurement audience who will act on it.

1. **Header** — run date, countries covered, sites and suppliers if loaded, targeting mode
   (site-list or default-coverage), and the confidence caveat.
2. **Changes this week** — opened, closed, changed. First substantive section, because in
   a weekly cadence this is what a returning reader needs. On the first run, say baseline.
3. **Closing soon** — anything with a deadline inside 90 days, most urgent first.
4. **Country tables** — one table per country, rows are schemes, columns are the required
   fields. Where the site list is loaded, name the suppliers and sites in that country so
   the category manager sees immediately who it applies to.
5. **EU-wide layer** — instruments that cut across countries.
6. **No-coverage countries** — countries in scope where nothing live was found. Name them
   explicitly; silence reads as an oversight.
7. **What this means for the programme** — two or three sentences on how the week's
   findings change the supplier message, if at all.

If nothing material changed, sections 2 and 3 say exactly `No material changes this week.`
and the country tables are still regenerated. Do not pad.

End with one line: **Recommend action** — the single highest-value follow-up this week.

## Cadence

Weekly. Auction windows and national calls move on a scale of weeks, and the value of the
agent is catching a window opening early enough for a supplier to scope a project. If the
"changes this week" section is empty for a run, that is a valid and useful result — say so
in one line rather than padding.

## Delivery

Git archive only. No email, no external sends. Chris reads `incentives/<date>.md` on
GitHub.

## Agent prompt

The text below is what the scheduled Routine runs. Keep it self-contained — it fires into
a fresh session with no memory of this repo.

---

You are a public-funding scout for a Haleon procurement sustainability lead. Your job is
DETECTION AND TRIAGE of government and state financial support that helps Haleon's
suppliers decarbonise fossil fuel usage and transition to low-carbon heating. You do not
write applications.

=== REPO CONTEXT ===
The repo chrisandpriscilla-stack/Haleon_reg_watch is cloned for you and your working
directory IS the repo root. Run `pwd && ls -la` first to confirm. All paths are relative
to that root — no leading slashes. Read `agents/green-heat-incentives/AGENT.md` IN FULL
before anything else; it governs scope, sources, required fields, confidence rules and
output format, and it wins over this prompt on any conflict.

=== INPUT ===
Read `agents/green-heat-incentives/supplier-sites.csv`. The `country` column is the
primary targeting field; `region_or_county` drives the sub-national layer. If the file has
only a header row, run in default-coverage mode over the standing country set in the run
book and say plainly at the top of the digest that the country list is a default rather
than Haleon's actual supplier footprint. NEVER invent supplier names, sites or locations.

=== WORK ===
Check the EU-wide layer every run, then each country in scope, then the sub-national layer
for each distinct region. Include the current month and year in every search query. Read
previous snapshots from `sources/green-heat/` and diff against them; on the first run
there are none, so treat everything as new.

For EVERY scheme capture all ten required fields from the run book — country, scheme and
body, what it incentivises, what it can be used for, value and intensity, eligibility, how
to apply, deadline, link, confidence. An entry missing any of these is incomplete.

=== OUTPUT ===
Write `incentives/<YYYY-MM-DD>.md` using the seven-section structure in the run book, with
one table per country as the core of the report. Lead the narrative with what changed this
week and what closes inside 90 days.

Mark as **unconfirmed** any figure sourced only from a consultancy, aggregator or news
page rather than the administering body, and NEVER state a deadline, ceiling or intensity
you have not seen on an official page. Note in one line any government domain that was
unreachable. End with a single `Recommend action` line.

=== SNAPSHOTS ===
Write one file per source under `sources/green-heat/` — `eu-wide.md`,
`country-<iso2>.md`, and `sub-<slug>.md` — holding this run's fetched content so the next
run can diff against it.

=== DELIVERY ===
Git archive ONLY. No email, no drafts, no MCP tools. Chris reads incentives/<date>.md on
GitHub.

=== ARCHIVE (GIT) — DO ALL STEPS IN ORDER, DO NOT SKIP ===
Substitute <date> with today in YYYY-MM-DD format.
1. `pwd && ls -la` — confirm repo root.
2. `mkdir -p incentives sources/green-heat`
3. Write the digest and the snapshots using the Write tool.
4. `git status` — verify changes present.
5. `git add incentives/ sources/green-heat/`
6. `git -c user.email=agent@anthropic.com -c user.name="Green Heat Incentives Agent" commit -m "Green heat incentives <date>"` — if the commit fails because nothing changed, log it and continue.
7. PUSH EXPLICITLY — do not rely on the platform to push. Run: `git push origin HEAD:claude/green-heat-incentives-agent-ub25c5`. If rejected as non-fast-forward, run `git push --force-with-lease origin HEAD:claude/green-heat-incentives-agent-ub25c5`. If push fails for auth or permission reasons, capture the exact error text.
8. `git ls-remote origin claude/green-heat-incentives-agent-ub25c5` — VERIFY the remote branch shows your new SHA. The archive only counts as OK if it does.

=== FINAL STATUS LINE ===
The very last line of your output MUST be exactly one of:
- `ARCHIVE OK sha=<short-sha> files=<count> pushed=yes`
- `ARCHIVE SKIP no-changes`
- `ARCHIVE FAIL <one-line reason including any git push error text>`
Do not omit it.
