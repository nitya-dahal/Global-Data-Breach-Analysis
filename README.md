# Global Data Breach Analysis

A Power BI dashboard analyzing 352 real-world data breaches from 2004–2022, built from a raw, messy public dataset. This project focuses heavily on data cleaning and standardization in Power Query before building the visuals.

## Files

| File | Description |
|---|---|
| `Global_Data_Breach_Analysis.pbix` | The Power BI report — open in Power BI Desktop |
| `df_1.csv` | Raw source data before cleaning |

## Data Source & Scope

352 breach records spanning 2004–2022, covering organization, breach year, records exposed, method of breach, and organization type.

## Dashboard

![Dashboard overview](images/dashboard-overview.png)

![Breach detail table with filters](images/dashboard-table-view.png)

At a glance: 352 total breaches, 307 confirmed, 13bn records exposed in total, averaging ~38M per breach. Breaches broken down by industry, method, and year, with slicers for Industry, Records Confidence, and Breach Year range.

## Data Cleaning (Power Query)

The raw CSV had several quality issues that were addressed before modeling:

**1. Dropped junk columns**
- `Unnamed: 0` — leftover pandas index column
- `Sources` — citation numbers (e.g. `[5][6]`), not usable in visuals

**2. `Breach Year` — fixed 3 range values**
Replaced `2019-2020` → `2019`, `2018-2019` → `2018`, `2014 and 2015` → `2014` so the column could be treated as a clean whole number.

**3. `Records Exposed` — the messiest column**
~26 non-numeric entries (`"unknown"`, `"tens of thousands"`, `"235 GB"`, compound strings like `"9,000,000 (approx) - basic booking, 2208 (credit card details)"`) plus 2 blanks. Rather than dropping these rows, the original text was preserved and split into three columns:
- `Records Exposed (Raw Text)` — original value, kept for audit/transparency
- `Records Exposed` — cleaned whole number (parsed or estimated where reasonable, null where truly unknown)
- `Records Confidence` — flags each row as `Confirmed`, `Estimated`, or `Unknown`, so the dashboard can distinguish hard numbers from estimates

**4. `Organization Type` — consolidated 70 raw values into 13 categories**
Original values had heavy overlap (`web` / `web service` / `web, tech`, `health` / `healthcare` / `government, healthcare`, etc.). Consolidated via a custom-column lookup table using the rule: *when a raw value lists multiple types, the first-listed type is treated as primary.*

Final categories: Government & Politics, Healthcare, Military & Defense, Banking & Finance, Technology & Software, Web Services & Social Media, Retail & Consumer Goods, Telecommunications, Media & Entertainment, Hospitality & Travel, Education, Data & Analytics Services, Other/Miscellaneous.

**5. `Breach Method` — consolidated 25 raw values into 7 categories**
Same lookup-table approach, standardizing inconsistent casing and compound values (`poor security` vs `Poor security` vs `poor security/inside job`) into: Hacked / External Attack, Poor Security / Misconfiguration, Insider Threat, Accidental Exposure, Lost / Stolen Device, Social Engineering, Unknown.

## Data Model

Single flat table (`df_1`) — no relationships needed, so no separate dimension/date tables. `Breach Year` is kept as a Whole Number rather than a Date type since only the year is meaningful here.

## DAX Measures

```dax
Total Records Exposed = SUM('df_1'[Records Exposed])

Total Breaches = COUNTROWS('df_1')

Avg Records Per Breach = DIVIDE([Total Records Exposed], [Total Breaches])

Confirmed Breaches = CALCULATE([Total Breaches], 'df_1'[Records Confidence] = "Confirmed")
```

## Tools

- Power BI Desktop (Power Query / M, DAX)

## Notes

- Some `Records Exposed` figures are estimates flagged via `Records Confidence`, not confirmed counts — check that column before treating totals as precise.
- Category consolidation involved judgment calls (e.g. QR code payment folded into Banking & Finance, gaming folded into Media & Entertainment) — see the mapping tables in the Power Query steps for the full logic.
