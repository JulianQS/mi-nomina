## Context

`index.html` is a single-file, no-build React 18 app. All business logic lives as plain JS functions above the component tree. The existing payroll logic (`calcPayroll`, `getEffectiveSalary`) already handles per-period salary lookup and transport subsidy eligibility — prima reuses both patterns.

Colombian law (CST Art. 306): prima de servicios = `(monthly_salary [+ transport_if_eligible]) × days_in_semester / 360`. Semesters use the 30-days/month convention (max 180 days each). If an employee joined mid-semester, days are prorated from `contract_start`. Transport subsidy is included in the base only for employees earning ≤ 2× minimum wage.

## Goals / Non-Goals

**Goals:**
- Pure calculation functions for prima amount per employee per semester.
- A UI panel showing both semesters of the current year with due dates and urgency highlighting.
- Prorated amounts for employees whose `contract_start` falls mid-semester.

**Non-Goals:**
- Tracking whether prima has actually been paid (no payment ledger in this change).
- Historical prima across past years.
- Aguinaldo or other bonuses.

## Decisions

**Where to render the prima panel**
Place it in the existing Payroll tab, below the quincena calculator section. Avoids adding a new top-level tab for what is a payroll-adjacent calculation. Alternative considered: separate tab — rejected as it fragments the payroll mental model.

**Salary to use for a full-semester employee**
Use `getEffectiveSalary(emp, semesterEndDate)` — the salary effective at the end of the semester, consistent with how `calcPayroll` works. For partial semesters (hire mid-period), still use the end-of-semester effective salary; only the days are prorated.

**Days calculation**
Days in semester = `min(semesterEndDay, today) - max(semesterStartDay, contractStartDay)` in 30-day months, capped at 180. This gives accrued-to-date prima when the semester is in progress, and full prima when it's complete.

**Urgency threshold**
Highlight due date red when ≤ 30 calendar days away. Mirrors the kind of visual cue the rest of the app uses for actionable items.

## Risks / Trade-offs

- [Salary changes within a semester] Using the end-of-semester salary is a simplification; strictly the prima base should reflect all wages earned. For the typical case (one salary the whole period), the result is exact. → Acceptable for this tool's scope; document in UI tooltip if needed.
- [Leap year / month-length edge cases] Using the 30-days/month convention (same as Colombian labor calculations) sidesteps calendar math complexity. → No mitigation needed; this is the legal convention.
