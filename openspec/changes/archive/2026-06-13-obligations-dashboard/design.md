## Context

`index.html` is a single-file, zero-dependency React app. All state lives in the `App` component and persists to `localStorage` under `mi_nomina_db`. Payroll (`calcPayroll`) and prima (`calcPrima`, `semesters`) calculations already exist as pure functions. The dashboard today is PTO-centric and there is no concept of cesantías, no merged view of upcoming payments, and no way to record an obligation as settled.

Note: the calculation functions (`calcCesantias`, `upcomingObligations`) and the redesigned `Dashboard` component were drafted alongside this proposal; the mark-as-paid persistence and UI action remain to be implemented. This design documents the whole change so the specs and code stay coherent.

## Goals / Non-Goals

**Goals:**
- Surface employer obligations (quincenas, prima, cesantías, intereses) as a dated, sorted, forward-looking agenda.
- Compute cesantías and intereses with statutory due dates, reusing the prima proration approach.
- Make the dashboard money-first; keep PTO present but secondary.
- Let the user mark an obligation paid so it leaves the agenda but is kept as history.

**Non-Goals:**
- No new tabs, backend, or dependencies.
- No automatic payment integration — marking paid is a manual user action.
- No editing of paid records beyond create (and optionally undo); full ledger management is out of scope.
- No change to existing quincena or prima calculation requirements.

## Decisions

**Reuse the 30-day-month proration for cesantías.** Cesantías mirror prima: `(salary + transport) × days / 360`. Using the existing `daysBetween30` helper keeps results consistent with prima and avoids a second date convention. Alternative considered: actual calendar days — rejected for inconsistency with the rest of the app.

**One generator merges all obligation types.** `upcomingObligations(emp, count)` scans quincena pay dates (~8 months ahead), prima due dates (this year + next), and cesantías/intereses (prior + current year), then filters past/pre-contract/paid items and sorts by date. A single source keeps the hero and the agenda list in sync — the hero is just `agenda[0]`.

**Quincena pay dates are the 15th and month-end.** These match the quincena bounds already used by `calcPayroll`. Prima (Jun 30) and a quincena (Jun 30) on the same day appear as two separate rows rather than a combined line — simpler and each is individually markable as paid.

**Paid records keyed by `type + due date`.** Obligations are not stored objects; they are recomputed each render. To mark one paid we need a stable identity. `type` plus the ISO due date is unique and deterministic, so a paid record `{ type, key, amount, paidDate }` can be matched against freshly generated items to filter them out. Alternative considered: storing full obligation snapshots — rejected as redundant since amounts are derivable.

**Persist paid records on the employee.** Add an optional `paid_obligations` array to each employee object, saved with the existing `mi_nomina_db` write. Absence of the field = nothing paid, so old exports load unchanged.

## Risks / Trade-offs

- [Recomputed amounts can drift from the amount shown when paid, e.g. after a retroactive salary edit] → The paid record stores the `amount` at time of payment, so history reflects what was actually settled rather than a recomputed figure.
- [A future salary change could alter a not-yet-paid obligation's amount] → Acceptable and arguably correct: unpaid projections should reflect current data.
- [Period-key collisions if two obligations of the same type share a due date] → Types are distinct per due date in practice (one quincena, one prima, etc.); the key includes type to disambiguate.
- [Agenda scan windows are bounded (8 months / this+next year)] → Sufficient for a 5-item list; widen the constants if longer horizons are ever needed.

## Migration Plan

- Additive schema change; no migration step required. Employees without `paid_obligations` are treated as having none.
- Rollback: removing the feature leaves `paid_obligations` as harmless unused data in stored JSON.
