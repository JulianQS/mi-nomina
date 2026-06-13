## 1. Cesantías calculation

- [x] 1.1 Add `cesantiasPeriod(year)` returning year bounds + due dates (intereses Jan 31, cesantías Feb 14 of Y+1)
- [x] 1.2 Add `calcCesantias(emp, year)` computing days (30-day months, prorated from contract_start, cap 360), cesantías, and intereses
- [x] 1.3 Verify against worked examples: full year at minimum wage → 2,000,000 cesantías / 240,000 intereses; mid-year hire prorates

## 2. Obligations agenda generator

- [x] 2.1 Add `upcomingObligations(emp, count)` merging quincenas (via `calcPayroll`), prima, cesantías, and intereses
- [x] 2.2 Exclude past-due and pre-contract items; sort ascending by date; limit to `count`
- [x] 2.3 Exclude obligations that have a matching paid record

## 3. Dashboard redesign

- [x] 3.1 Replace PTO hero with a "Próximo pago" hero (amount, label, date, countdown with urgency colors)
- [x] 3.2 Add empty state for when no obligations are upcoming
- [x] 3.3 Add snapshot stat row (vacaciones, salario mensual, próxima acumulación PTO)
- [x] 3.4 Add "Próximas obligaciones" agenda card with per-type color, date, period, countdown, amount
- [x] 3.5 Keep "Últimas ausencias" below the agenda

## 4. Mark obligation as paid

- [x] 4.1 Define the paid-record shape `{ type, key (type + due date), amount, paidDate }`
- [x] 4.2 Add optional `paid_obligations` array to the employee schema; ensure save/load round-trips it
- [x] 4.3 Add a mark-as-paid action to each agenda row that appends a paid record via `updateEmp`
- [x] 4.4 Filter paid obligations out of `upcomingObligations` using the period key
- [x] 4.5 (Optional) Show paid obligations as history and allow undo

## 5. Documentation & verification

- [x] 5.1 Update README data schema with `paid_obligations`
- [x] 5.2 Update CLAUDE.md (new sections/functions: cesantías, obligations agenda)
- [x] 5.3 Manually verify in browser: agenda order, mark-as-paid removes item and persists across reload

## 6. Historical wages

- [x] 6.1 Add `WAGE_HISTORY` table (2024, 2025, 2026) with `minWage` and `transport` per year
- [x] 6.2 Add `getWagesForYear(year)` helper with fallback to most recent year
- [x] 6.3 Update `getEffectiveSalary` to resolve min wage and transport from `WAGE_HISTORY[year]`; return `transportMonthly` on result
- [x] 6.4 Update `calcPayroll`, `calcPrima`, `calcCesantias` to use `sal.transportMonthly` instead of global `MONTHLY_TRANSPORT`
- [x] 6.5 Update Quincena "Valores fijos" reference table to use year-appropriate values and label

## 7. Quincena proration & historical access

- [x] 7.1 Prorate salary and transport in `calcPayroll` when `contract_start` falls within the quincena period (`daysWorked / 30`)
- [x] 7.2 Return `prorated`, `daysWorked`, `daysInPeriod` from `calcPayroll`
- [x] 7.3 Show "Prorrateado · N/M días" badge in quincena UI when prorated
- [x] 7.4 Add 2024 to the year dropdown in the Quincena tab
- [x] 7.5 Add "Cesantías" option to the period dropdown (months 1–2 only), rendering CesantiasAnualView — a full calculation card with formula strings, "Base del cálculo" table, and urgency highlighting — mirroring the PrimaView pattern used for June/December
