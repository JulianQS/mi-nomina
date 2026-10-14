## Why

The app already handles quincena payroll but has no visibility into **prima de servicios** — the mandatory semi-annual bonus Colombian employers must pay by June 30 and December 20. Employers need to know the exact amount owed per employee and how much has accrued so far mid-semester, before those deadlines arrive.

## What Changes

- Add a **Prima** section to the Payroll tab (or as a new sub-tab) showing accrued and projected prima for each semester per employee.
- Display two rows per employee per year: 1st semester (Jan–Jun) and 2nd semester (Jul–Dec), each with days worked, accrued amount, and due date.
- Highlight upcoming due dates (June 30 / December 20) when within 30 days.

## Capabilities

### New Capabilities

- `prima-calculation`: Compute prima de servicios for a given employee and semester. Formula: `(monthly_salary + transport_subsidy_if_applicable) × calendar_days_in_semester / 360`. Salary used is the effective salary per `salary_history` for the period. Transport subsidy is included only for employees earning ≤ 2× minimum wage.
- `prima-view`: UI component that shows per-employee prima breakdown (semester, days, accrued amount, due date) and highlights imminent payment deadlines.

### Modified Capabilities

<!-- No existing specs are changing. -->

## Impact

- `index.html` only — no backend, no new dependencies.
- New pure functions alongside existing `calcPayroll` / `getEffectiveSalary`.
- New React component rendered inside the existing Payroll tab or alongside it.
