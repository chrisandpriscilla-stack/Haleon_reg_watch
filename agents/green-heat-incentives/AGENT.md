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

## Priority technologies

Four technologies are the priority lens. Tag every scheme with which of them it supports,
using these labels:

- **HP** — heat pumps, including high-temperature and industrial process heat pumps
- **BIO** — biomass, in all forms including gasification and densified fuel
- **BGS** — biogas, biomethane, bio-CNG and anaerobic digestion
- **STOR** — heat storage and thermal energy storage

Schemes supporting other low-carbon heat routes (electric and electrode boilers, solar
thermal, waste-heat recovery, heat network connection) remain **in scope and should still
be reported** — tag them `OTHER`. The four priorities determine ordering and emphasis, not
what gets excluded. Where a scheme covers several, list all applicable tags.

A caution to carry into every biomass and biogas entry: sustainability and air-quality
restrictions vary sharply by country, and biomass in particular raises deforestation,
particulates and land-use concerns. Where a scheme carries feedstock or emissions
conditions, say so in the eligibility column. A supplier switching to biomass in breach of
local air rules is a worse outcome than no switch at all.

## Geographic scope

| Region | Coverage |
| --- | --- |
| **EU** | The EU-wide layer plus member-state programmes. Germany, France, Netherlands, Spain, Italy and Poland are the standing priority set; extend to other member states when the site list requires |
| **India** | National (MNRE, BEE) plus state schemes where `region_or_county` is populated |
| **China** | National (NDRC, MIIT) plus provincial funds |
| **Brazil** | Federal (BNDES, MME) plus state programmes |
| **Pakistan** | Federal (AEDB, Ministry of Energy) and State Bank financing schemes |
| **South East Asia** | Thailand, Vietnam, Indonesia, Malaysia, Philippines, Singapore |
| **Turkey** | National (KOSGEB, Ministry of Energy and Natural Resources) and World Bank-backed programmes |
| **Global** | Multilateral and cross-border finance — see below |
| **Legacy coverage** | United Kingdom, Ireland and the United States remain covered from the earlier scope. Do not drop them |

The **Global layer** is checked every run regardless of the site list, because these
instruments reach across most of the emerging-market geographies above and are often the
only meaningful money available in them: Climate Investment Funds (notably the Industry
Decarbonization Program), Green Climate Fund, World Bank and IFC, Asian Development Bank,
Inter-American Development Bank, EBRD, plus Just Energy Transition Partnership packages
where they exist.

Note how these differ from the EU and UK instruments: multilateral finance usually arrives
as **concessional loans, guarantees and technical assistance channelled through local
banks**, not as a grant a supplier applies for directly. The practical route is almost
always a participating financial institution, so the "how to apply" field must name that
route rather than pointing at the fund.

## Inputs

`supplier-sites.csv` in this directory drives which countries are covered. One row per
supplier **site**, not per supplier — a supplier with plants in three countries generates
three rows, because incentives are located where the asset is.

| column | meaning |
| --- | --- |
| `site_id` | stable key, your choice of format |
| `supplier_name` | legal or trading name |
| `site_name` | plant identifier |
| `country` | **the primary targeting field** |
| `region_or_county` | secondary targeting — Indian or US state, Chinese province, Brazilian state, German Land, Spanish autonomous community, UK county |
| `postcode_or_zip` | narrows to local schemes |
| `current_heat_source` | e.g. natural gas steam boiler, LPG, heavy fuel oil, coal |
| `annual_heat_demand_mwh` | sizes the opportunity; blank if unknown |
| `annual_heat_tco2e` | Scope 1 for the supplier, Scope 3 Cat. 1 for Haleon |
| `haleon_category` | procurement category |
| `haleon_annual_spend_gbp` | prioritisation weight |
| `category_manager` | who owns the relationship and carries the message |
| `notes` | anything else |

See `supplier-sites.example.csv` for the shape. Only `country` is strictly required;
`region_or_county` unlocks the sub-national layer, and the rest improves prioritisation.

**If `supplier-sites.csv` has no data rows**, the agent runs in *default-coverage mode*
across the full geographic scope above and states plainly at the top of the digest that
the country list is a default, not Haleon's actual supplier footprint. It must never
invent supplier names, sites or locations.

## Scope — what counts

In scope: capital grants and subsidies for fuel switching off gas, oil, LPG and coal
process heat; operating subsidies and contracts-for-difference that close the running-cost
gap; competitive auctions for industrial heat; tax instruments with a real subsidy element
including accelerated depreciation, investment allowances and import-duty exemptions; soft
loans, credit guarantees and interest rebates; heat network connection funding;
feasibility studies, energy audits and free diagnostic visits.

Out of scope: domestic and household schemes, transport and EV funding, pure renewable
*electricity* funding (that is the Schneider Energize track — note and move on), R&D-only
calls with no deployment pathway, and anything a supplier cannot apply for directly or as
a named partner.

## Required fields for every scheme

Every entry must answer all of these. An entry missing any is incomplete:

1. **Country** and, where relevant, sub-national geography
2. **Scheme name and administering body**
3. **Technology tags** — HP, BIO, BGS, STOR, OTHER
4. **What it incentivises** — the behaviour or outcome being paid for
5. **What it can be used for** — eligible costs and technologies, concretely
6. **Value** — grant ceiling, intensity as a % of project cost, subsidy rate, guarantee
   percentage or interest rebate, plus any SME uplift
7. **Eligibility** — who can apply, SME-only restrictions, sector restrictions, whether a
   supplier to a large corporate qualifies, and any feedstock or emissions conditions
8. **How to apply** — the mechanism: rolling application, competitive call, auction, tax
   return, or via a participating financial institution or accredited installer; and the
   practical first step
9. **Deadline** — a date, "rolling", or the expected opening of the next window
10. **Link** to the administering body
11. **Confidence** — see below

## Confidence discipline

Never state a deadline, grant ceiling or intensity you have not seen on an official page.
Where a scheme is sourced only from a consultancy, aggregator or news page, mark it
**unconfirmed** and say so in the row. Suppliers make investment decisions on this; a wrong
deadline is worse than an absent one.

This matters more in the emerging-market geographies than in the EU. Announced programmes
frequently run years behind their headline, and headline fund sizes are often the total
programme envelope rather than money a supplier can access. Where a figure is a programme
envelope rather than an available grant, say which it is.

### Verification constraint

As at the baseline runs, the execution environment's network policy blocked direct fetches
of `www.gov.uk`, `assets.publishing.service.gov.uk`, `www.gov.scot` and
`businesswales.gov.wales` (CONNECT refused with 403). Retry the direct fetch each run — the
policy may change — and where it still fails, mark the affected figures **unconfirmed**
rather than dropping them. Note the blocked domains in one line in the digest. A blocked
fetch is not a reason to skip a source.

## Sources to check each run

**Global layer** (every run): Climate Investment Funds, Green Climate Fund, World Bank,
IFC, ADB, IDB, EBRD, JETP packages.

**EU layer** (every run): Innovation Fund, especially the Heat Auction series (DG CLIMA and
CINEA); Modernisation Fund; LIFE; EIB mechanisms; live NRRP allocations.

**National agencies** per country in scope: DESNZ and gov.uk (UK), SEAI (Ireland), BMWE and
KEI (Germany), ADEME (France), RVO (Netherlands), IDAE (Spain), GSE and MASE (Italy),
NFOŚiGW (Poland), DOE and IRS (United States), MNRE and BEE (India), NDRC and MIIT (China),
BNDES and MME (Brazil), AEDB (Pakistan), DEDE and BOI (Thailand), MOIT (Vietnam), ESDM
(Indonesia), MGTC and the GTFS administrator (Malaysia), KOSGEB and MENR (Turkey).

**Sub-national layer** for each distinct `region_or_county`.

Always include the current month and year in search queries — windows turn over fast and
cached summaries go stale within weeks.

## State handling

Read previous snapshots from `sources/green-heat/`. On the first run these are absent — skip
the diff and say so. Thereafter surface **changes**: newly opened windows, deadlines now
inside 90 days, schemes closed or exhausted, changes to value or eligibility, and new
schemes in a country that previously had none. Do not re-list an unchanged scheme in full;
one line confirming it is still open is enough. The full country tables are regenerated each
run so the digest stands alone, but the narrative sections lead with what moved.

## Output — `incentives/<YYYY-MM-DD>.md`

UK English, for a senior procurement audience who will act on it.

1. **Header** — run date, regions and countries covered, sites and suppliers if loaded,
   targeting mode, and the confidence caveat.
2. **Changes this week** — opened, closed, changed. First substantive section, because in a
   weekly cadence this is what a returning reader needs.
3. **Closing soon** — anything with a deadline inside 90 days, most urgent first.
4. **Global layer** — multilateral instruments cutting across geographies.
5. **Country tables** — grouped by region, one table per country, rows are schemes, columns
   are the required fields including technology tags. Where the site list is loaded, name
   the suppliers and sites in that country.
6. **No-coverage countries** — countries in scope where nothing live was found. Name them
   explicitly; silence reads as an oversight.
7. **What this means for the programme** — two or three sentences on how the week's findings
   change the supplier message, if at all.

If nothing material changed, sections 2 and 3 say exactly `No material changes this week.`
and the country tables are still regenerated. Do not pad.

End with one line: **Recommend action** — the single highest-value follow-up this week.

## Cadence

Weekly. If the "changes this week" section is empty for a run, that is a valid and useful
result — say so in one line rather than padding.

## Delivery

Git archive only. No email, no external sends. Chris reads `incentives/<date>.md` on GitHub.

## Agent prompt

The text below is what the scheduled Routine runs. Keep it self-contained — it fires into a
fresh session with no memory of this repo.

---

You are a public-funding scout for a Haleon procurement sustainability lead. Your job is
DETECTION AND TRIAGE of government and state grants, subsidies and incentives that could
assist Haleon's suppliers implement low-carbon heating solutions. You do not write
applications.

=== REPO CONTEXT ===
The repo chrisandpriscilla-stack/Haleon_reg_watch is cloned for you and your working
directory IS the repo root. Run `pwd && ls -la` first to confirm. All paths are relative to
that root — no leading slashes. Read `agents/green-heat-incentives/AGENT.md` IN FULL before
anything else; it governs scope, geography, priority technologies, required fields,
confidence rules and output format, and it wins over this prompt on any conflict.

=== PRIORITY TECHNOLOGIES ===
Tag every scheme: HP (heat pumps incl. high-temperature industrial), BIO (biomass), BGS
(biogas/biomethane/anaerobic digestion), STOR (heat and thermal storage), OTHER (electric
boilers, solar thermal, waste-heat recovery, heat network connection). The four priorities
drive ordering and emphasis; OTHER schemes are still reported, not excluded. For BIO and BGS
entries, always state any feedstock, air-quality or sustainability conditions in the
eligibility field.

=== GEOGRAPHY ===
Cover EU (EU-wide layer plus Germany, France, Netherlands, Spain, Italy, Poland), India,
China, Brazil, Pakistan, South East Asia (Thailand, Vietnam, Indonesia, Malaysia,
Philippines, Singapore), Turkey, and the Global multilateral layer. Retain the legacy UK,
Ireland and United States coverage — do not drop it.

Check the Global layer EVERY run — Climate Investment Funds, Green Climate Fund, World Bank,
IFC, ADB, IDB, EBRD, and JETP packages — because it is often the only meaningful money in the
emerging-market geographies. Remember these usually arrive as concessional loans, guarantees
or technical assistance channelled through local participating banks rather than as direct
grants, so "how to apply" must name that route.

=== INPUT ===
Read `agents/green-heat-incentives/supplier-sites.csv`. The `country` column is the primary
targeting field; `region_or_county` drives the sub-national layer. If the file has only a
header row, run in default-coverage mode across the full geographic scope and state plainly
at the top of the digest that the country list is a default rather than Haleon's actual
supplier footprint. NEVER invent supplier names, sites or locations.

=== WORK ===
Include the current month and year in EVERY search query. Read previous snapshots from
`sources/green-heat/` and diff against them; on the first run there are none, so treat
everything as new.

For EVERY scheme capture all eleven required fields from the run book — country and
sub-national geography, scheme and body, technology tags, what it incentivises, what it can
be used for, value, eligibility, how to apply, deadline, link, confidence. An entry missing
any of these is incomplete.

=== OUTPUT ===
Write `incentives/<YYYY-MM-DD>.md` using the seven-section structure in the run book, with
country tables grouped by region as the core of the report. Lead the narrative with what
changed this week and what closes inside 90 days.

Mark as **unconfirmed** any figure sourced only from a consultancy, aggregator or news page
rather than the administering body, and NEVER state a deadline, ceiling or intensity you have
not seen on an official page. Where a headline figure is a total programme envelope rather
than money a supplier can access, say which it is. Note in one line any government domain
that was unreachable. End with a single `Recommend action` line.

=== SNAPSHOTS ===
Write one file per source under `sources/green-heat/` — `global.md`, `eu-wide.md`,
`country-<iso2>.md`, `sub-<slug>.md` — holding this run's fetched content so the next run can
diff against it.

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
7. PUSH EXPLICITLY to main — do not rely on the platform to push. The archive accumulates on `main` (see the repo README), so each run starts from the latest `main`, appends its digest and overwrites its snapshots. Run: `git push origin HEAD:main`. If rejected as non-fast-forward (main moved since your clone — e.g. the reg-watch agent pushed), run `git pull --rebase origin main` and then `git push origin HEAD:main` again. Never force-push `main`. If push fails for auth or permission reasons, capture the exact error text.
8. `git ls-remote origin main` — VERIFY the remote `main` shows your new SHA. The archive only counts as OK if it does.

=== FINAL STATUS LINE ===
The very last line of your output MUST be exactly one of:
- `ARCHIVE OK sha=<short-sha> files=<count> pushed=yes`
- `ARCHIVE SKIP no-changes`
- `ARCHIVE FAIL <one-line reason including any git push error text>`
Do not omit it.
