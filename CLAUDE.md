# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Mi Nómina is a zero-dependency HR tracker for small teams in Colombia. It has two parts:

- **`index.html`** — the entire app. A single-file React 18 app (JSX compiled by Babel Standalone at runtime, styled with Tailwind CDN). Opens directly in a browser — no build step, no server. Data lives in `localStorage` under the key `mi_nomina_db`.

## Running the app

```bash
open index.html          # macOS — opens in default browser
python -m http.server    # or serve locally if CORS is a concern
```

## Architecture of `index.html`

The file is a single ~1000-line script block. Sections are marked with `// ── SECTION ───` comments:

| Section | Purpose |
|---|---|
| CONSTANTS | `MONTHLY_MIN_WAGE`, `MONTHLY_TRANSPORT`, `MONTHLY_ACCRUAL` — update each January |
| STORAGE / MIGRATION | `loadDB()` / `saveDB()` — reads `mi_nomina_db` from localStorage, migrates legacy `pto_db` format |
| DATE HELPERS | Pure functions for month-end dates and accrual counting |
| PTO CALCULATIONS | `calcAccrued`, `calcTaken`, `calcBalance` — all take an `emp` object |
| SALARY HELPERS | `getEffectiveSalary(emp, dateStr)` — walks `salary_history` to find the salary in effect on a given date |
| PAYROLL | `calcPayroll(emp, year, month, half)` — returns a full quincena breakdown |
| FORMATTING | `fmtCOP`, `fmtDate`, etc. |
| SHARED COMPONENTS | `Card`, `Badge`, `StatCard`, `InputField`, `SelectField`, `BottomSheet` |
| Feature components | `EnrollForm`, `SalaryForm`, `PTOForm`, `PTOView`, `PayrollView`, `EmployeeList` |
| App root | `App` — holds `db` state, renders tab navigation |

State is kept entirely in the `App` component. There is no router — tabs (`pto`, `payroll`, `employees`) are controlled by a `tab` state variable.

## Colombia payroll rules encoded in the app

- Monthly min wage: `MONTHLY_MIN_WAGE = 1_750_905` COP (2026)
- Transport subsidy: `MONTHLY_TRANSPORT = 249_095` COP/month — only for employees earning ≤ 2× min wage
- Social security deduction: 8% of quincena salary (health 4% + pension 4%)
- PTO accrual: 1.25 days/month (15 days/year), accrued at end of each month

## Updating for a new year

Change two constants near the top of the `<script>` block and add the new year to the quincena year dropdown (search for `2025,2026,2027`).

## Data schema

See README.md for the full JSON shape. Key points:
- `salary_history` is an array of dated entries; `getEffectiveSalary` picks the most recent entry on or before the target date.
- `is_minimum: true` means the amount is ignored and `MONTHLY_MIN_WAGE` is used instead, so minimum-wage employees auto-update when the constant changes.
- `pto_events` uses negative days (absences); accrual is computed separately.
