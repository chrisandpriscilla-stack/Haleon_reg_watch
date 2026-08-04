# Haleon_reg_watch

Automated watch agents supporting Haleon procurement sustainability. Each agent runs on
a schedule, writes a dated digest to the repo, and delivers via the git archive only —
there is no email step.

## Agents

### Scope 3 regulatory briefing — weekly

Tracks GHG Protocol (Scope 2, Scope 3, Actions and Market Instruments), SBTi Corporate
Net-Zero Standard V2, EU ETS, CBAM and Fit for 55 for changes bearing on the Valle 4
Spain vPPA, the Texas solar PV deal, the REV Scope 3 instrument and the 42%-by-2030
Scope 3 target.

- Digests: `digests/<date>.md`
- Snapshots for diffing: `sources/<slug>.md`

### Green heat incentives — monthly

Finds public funding that helps suppliers switch process heat off fossil fuels, mapped
to the counties where those suppliers actually run heat assets. Detection and triage
only; it does not write applications. Feeds the supplier-specific "value-at-stake"
one-pager used by category managers in QBRs and tenders.

- Run book: `agents/green-heat-incentives/AGENT.md`
- **Input you maintain:** `agents/green-heat-incentives/supplier-sites.csv` — one row per
  supplier *site*. `country` and `region_or_county` drive the search; everything else
  improves prioritisation. See `supplier-sites.example.csv` for the shape. While this
  file is empty the agent runs in jurisdiction-default mode and covers the UK national
  and devolved layer only.
- Digests: `incentives/<date>.md`
- Snapshots for diffing: `sources/green-heat/<slug>.md`
