# Tewkesbury-Borough-Council-Data-Analysis

Turns 12 monthly council spending data files from a UK local council into one clean, reconciled dataset, then analyses where the money went.

3,634 raw rows from 12 messy monthly files were cleaned into 3,629 rows totaling **£73,544,974.55**, reconciled to the raw files with a difference of £0.

## Skills used in this project
|Skill|Where it shows up|
|---|---|
|Excel data cleaning and financial analysis|Power Query cleaning, PivotTables, spend by supplier, department and expense type|
|Data validation and quality checks|Row and total reconciliation, duplicate analysis, a documented issue log|
|Business data analysis and reporting|Key findings and recommendations below|
|Excel dashboards and KPI reporting|PivotCharts and slicers (Payee Type, Department)|

## Dataset
• Source: Tewkesbury Borough Council, "Local Spending Data", January to December 2025, published as one file per month under the Local Government Transparency Code (https://tewkesbury.gov.uk/about-the-council/requests-for-information/spending-and-transparency/, Open Government License)
• Size: 12 files, 3634 rows, 10 columns (Body, Post Code, Service Area, Categorization, Expense Type, Date, Transaction Number, Department, Amount, Supplier Name, Supplier Number)
• Tools: Excel, Power Query, PivotTables.

## Workflow
1. Loaded the 12 monthly csv files as separate Power Query queries
2. Standardized column headers and data types in each query (see cleaning log below)
3. Appended the 12 queries into one table
4. Removed completely blank rows
5. Split the table into invoice rows and bank-type rows. Checked invoice rows for exact duplicates, and kept bank-type rows untouched
6. Added a `Payee Type` column to separate payees that should not be ranked as ordinary suppliers
7. Reconciled the clean table to the raw files
8. Built PivotTables and charts on the clean table

## Cleaning log
| Issue                                                                                                            | Where                      | Rows / £ affected           | Decision                                                                                                                                                                            |
| ---------------------------------------------------------------------------------------------------------------- | -------------------------- | --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Date and Amount stored as text (£ symbols, commas, dd/mm/yyyy dates)                                             | All months                 | All rows                    | Converted to Date and Decimal/Currency using the English (United Kingdom) locale, so day and month are read correctly                                                               |
| Header mismatch: `Amount` vs `Amount` (extra spaces)                                                             | February, April            | 585 rows, £14.65M           | Trimmed column names before appending. Without this, Amount was blank for these rows and the total was understated by about 20%                                                     |
| Decimal point lost during conversion (1,376,611.56 became 137,661,156)                                           | Whole table                | Every amount with pence     | set the workbook locale to English (United Kingdom). Verified against raw totals                                                                                                    |
| Completely blank rows                                                                                            | June                       | 5 rows                      | Removed                                                                                                                                                                             |
| Missing date                                                                                                     | October (HM Land Registry) | 1 row, £876                 | Kept the row and left the date null. Flagged in the validation log. Do not invent a date                                                                                            |
| Date outside its file's month (21/08/2025 in the June file)                                                      | June                       | 1 row, £2,990               | Kept as in source and flagged                                                                                                                                                       |
| Exact-duplicate rows                                                                                             | Across files               | 198 rows, £118,800          | **Kept.** All are separate £600 grant payments to different (redacted) individuals. Among invoice rows there are no exact duplicates [confirm 0 rows removed in your NonBank query] |
| Supplier Number is null                                                                                          | All months                 | 501 rows                    | Kept as null. They fall into two groups: 354 redacted grant payments and 147 bank, card and direct-debit items. Filling them would invent IDs                                       |
| Supplier name variants and spacing (double spaces, `HM LAND REGISTRY` vs `HM LAND REGISTR`, Travelodge variants) | Various                    | 23 names with double spaces | Limitation as it is not in the date range. Not included in the PivotTable amount column total but is separate.                                                                      |

## Payee Type

Some rows in a spending file are not ordinary suppliers. This column keeps them out of supplier rankings.

|Payee Type|Rule|Rows|Amount|
|---|---|---|---|
|Supplier|Everything else|2,992|£72,439,153.70|
|Redacted - grant payment|Supplier Name is "Redacted Personal Information" and Transaction Number is "Bank"|354|£219,490.00|
|Redacted - supplier number held|Supplier Name is "Redacted Personal Information" with a real Transaction Number|136|£361,437.62|
|Bank / card / direct debit|Transaction Number is "Bank", name not redacted|147|£524,893.23|
|**Total**||**3,629**|**£73,544,974.55**|

## Validation and reconciliation

| Check            | Raw files      | Clean table                          | Difference                    |
| ---------------- | -------------- | ------------------------------------ | ----------------------------- |
| Row count        | 3,634          | 3,629                                | 5 (blank rows, June)          |
| Total amount     | £73,544,974.55 | £73,544,974.55                       | £0.00                         |
| Payee Type split | n/a            | 2,992 + 354 + 136 + 147 = 3,629 rows | Reconciles to the grand total |

## Key findings

All figures use the 3,629 clean rows.

1. **Spend is highly concentrated.** The top 10 suppliers account for £58.5M, which is **80.7%** of supplier spend, out of about 530 payee names. The largest, Gloucestershire County Council, alone is £34.3M (**47.4%**).
2. **Most of the top 10 are not ordinary suppliers.** Seven of the ten are other public bodies (county, district and parish councils, the police commissioner, HMRC). The county council's payments are mostly "Payments Made" (£27.0M) and capital grants (£6.8M), so they are transfers, not purchases.
3. **The commercial picture is much smaller.** The largest commercial supplier is Ubico Limited (£7.0M). Dennis Eagle (£0.70M) and Casepak (£0.67M) follow, and the rest are under £1M each.
4. **Finance dominates by department.** Finance is £42.4M, **57.7%** of all spend. The top 10 departments cover 95.8% of spend (of 30 departments).
5. **"Payments Made" is a catch-all.** It accounts for £44.2M, **60.0%** of spend, and £38.6M of Finance's spend is in this category. The top 10 expense types cover 94.2% of spend (of 84 types), but the largest tells the reader little about what the money bought.

## Recommendations

1. **Report transfers to public bodies separately from procurement spend.** Mixing them makes supplier concentration look far higher than it is for commercial contracts.
2. **Use more specific expense categories for large payments.** A catch-all covering 60% of spend limits what any analysis can show.
3. **Standardise supplier names and IDs at source,** and make Date mandatory. Name variants split spend across rows, and one payment has no date.
4. **Keep redacted payments identifiable.** 136 redacted rows carry a supplier number and can be analysed. The 354 grant payments cannot be told apart.

## Limitations and next steps

- **One row has no date (£876)**, so it appears in a "before 2025" group in date-grouped PivotTables.
- **Supplier name variants may still split some suppliers across rows**. Performed a merge based on exact matches
- **Possible duplicate payments to review** : 145 invoice rows (about £3.0M) share the same supplier, amount and date as another row, often with consecutive transaction numbers. They may be legitimate, so none were removed. A review register would be the next step.
- **Redacted individual payments cannot be analyzed at payee level**.
