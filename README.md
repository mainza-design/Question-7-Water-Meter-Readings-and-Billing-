# Q7 – Water Meter Readings and Billing (schema `q7`)

**Purpose:** bill water usage with a two-band tariff: first 10 units at K5, units above 10 at K8.

## Objects
| Part | Object | What it does |
|---|---|---|
| (a) | `customers`, `readings`, `bills`, `receivable_snapshot` | non-negative meter values, month-start `reading_month`, `amount_paid` between 0 and `charge`; 6 readings covering 0, exactly 10 and >10 units |
| (b) | `fn_water_charge(units)` | tiered charge rounded to 2 decimals; rejects NULL/negative |
| (c) | `pr_generate_bill(reading, due)` | derives units, calls the function, inserts bill with `amount_paid = 0`; rejects unknown reading or duplicate bill |
| (d) | `trg_readings_check` | BEFORE INSERT OR UPDATE; rejects `current < previous` so the row is never stored |
| (e) | `v_overdue_bills` | customer, due date, outstanding amount for unpaid bills past due |
| (f) | `idx_bills_unpaid_due` | partial index on `due_on WHERE amount_paid < charge` |
| (g) | `pr_refresh_receivables()` + job `q7_refresh_receivables` | upsert today's total unpaid; cron `30 0 * * *` |

## Tariff check
0 → K0 · 10 → K50 · 12 → K66 · 15 → K90
