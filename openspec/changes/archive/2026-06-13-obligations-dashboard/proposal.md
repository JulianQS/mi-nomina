## Why

The dashboard is vacation-centric: its headline is the PTO balance, and three of its four stats are PTO. The thing an employer actually needs day-to-day — *what do I owe, and when* — is buried inside the Quincena tab and requires manually picking a year/month/half. There are no payment reminders, and two of the obligations people most often forget — **cesantías** (due Feb 14) and **intereses sobre cesantías** (due Jan 31) — are not computed at all. Even with reminders, a list is only useful if you can clear what you've already paid, otherwise stale items pile up and get ignored.

## What Changes

- Add **cesantías** and **intereses sobre cesantías** calculations (annual, prorated from `contract_start`, with their statutory due dates). Add a reminder card in the Quincena tab for January and February.
- Add an **upcoming-obligations** generator that merges every dated employer obligation — quincenas, prima, cesantías, intereses — into one chronologically sorted, forward-looking list.
- **Redesign the dashboard** around money: a "Próximo pago" hero (next amount + due date + countdown), a 2-column snapshot (vacaciones + salario mensual), and a "Próximas obligaciones" agenda. Vacation balance is demoted from headline to one stat; all UI uses "Vacaciones", not "PTO". **BREAKING** (UI only): the dashboard layout and stat set change; no data is lost.
- Add the ability to **mark an obligation as paid**, recording it (type, period, due date, amount, date paid) so it drops off the agenda but remains viewable as history.
- Add a **historical wage table** (`WAGE_HISTORY`) so all payroll, prima, and cesantías calculations use the correct SMLV and transport subsidy for the year being computed, not the current year's constants.
- **Prorate quincena salary and transport** when an employee's `contract_start` falls mid-quincena, with a UI badge showing the proration ratio.

## Capabilities

### New Capabilities

- `cesantias-calculation`: Compute cesantías and intereses sobre cesantías for an employee and calendar year. Cesantías = `(monthly_salary + transport_if_eligible) × days_worked / 360`; intereses = `cesantias × days_worked × 0.12 / 360`. Days prorated from `contract_start`; salary taken from `salary_history` effective at year end. Exposes statutory due dates: intereses by Jan 31, cesantías deposit by Feb 14 of the following year.
- `obligations-agenda`: Merge all dated obligations (quincenas via `calcPayroll`, prima, cesantías, intereses) into one sorted, forward-looking list, excluding any obligation already recorded as paid. Defines the paid-record shape (type, period key, due date, amount, paid date) and the rule that marking an obligation paid removes it from the agenda while preserving it as history.
- `obligations-dashboard`: Dashboard UI built on the agenda — a "Próximo pago" hero with countdown, a 2-column snapshot (vacaciones + monthly salary), the "Próximas obligaciones" list with per-item type/date/amount and a mark-as-paid action, paid history with undo, and recent absences.
- `historical-wages`: `WAGE_HISTORY` table and `getWagesForYear(year)` helper. `getEffectiveSalary` resolves minimum wage and transport from the period's year; returns `transportMonthly` so all callers use the correct statutory value.
- `payroll-proration`: `calcPayroll` prorates salary and transport by `daysWorked / 30` when `contract_start` falls within the quincena period. Returns `prorated`, `daysWorked`, `daysInPeriod` for UI display. Year dropdown in Quincena tab extended to include 2024.

### Modified Capabilities

<!-- No existing specs change requirements. prima-calculation is reused as-is by the agenda. -->

## Impact

- `index.html` only — no backend, no new dependencies.
- New pure functions (`calcCesantias`, `upcomingObligations`, `getWagesForYear`, `obligationKey`) alongside updated `calcPayroll` / `calcPrima` / `getEffectiveSalary`.
- Rewritten `Dashboard` component.
- Data schema gains an optional `paid_obligations` array per employee; persisted in `localStorage` under `mi_nomina_db`. Backward compatible — absence of the field means nothing is marked paid.
