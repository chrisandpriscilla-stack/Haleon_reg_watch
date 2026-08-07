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

### Green heat incentives — weekly

Identifies government and state incentives, grants, subsidies and other financial
support that help suppliers decarbonise fossil fuel usage and transition to low-carbon
heating. For every scheme it captures what it incentivises, what it can be used for,
value and intensity, eligibility, how to apply and the deadline — presented as a table
by country. Detection and triage only; it does not write applications. Feeds the
supplier-specific "value-at-stake" one-pager used by category managers in QBRs and
tenders.

Priority technologies are heat pumps (**HP**), biomass (**BIO**), biogas (**BGS**) and
heat storage (**STOR**); every scheme is tagged, and other low-carbon heat routes are
tagged **OTHER** and still reported.

Geographic scope: EU (EU-wide layer plus Germany, France, Netherlands, Spain, Italy,
Poland), India, China, Brazil, Pakistan, South East Asia, Turkey, and a Global
multilateral layer (CIF, GCF, World Bank/IFC, ADB, IDB, EBRD, JETP), with UK, Ireland
and US coverage retained.

- Run book: `agents/green-heat-incentives/AGENT.md`
- **Input you maintain:** `agents/green-heat-incentives/supplier-sites.csv` — one row per
  supplier *site*. `country` is the primary targeting field and the only required one;
  `region_or_county` unlocks the sub-national layer (US states, German Länder, Spanish
  autonomous communities, UK counties). See `supplier-sites.example.csv` for the shape.
  While the file is empty the agent runs in default-coverage mode over a standing set of
  countries rather than Haleon's actual footprint.
- Digests: `incentives/<date>.md`
- Snapshots for diffing: `sources/green-heat/<slug>.md`
