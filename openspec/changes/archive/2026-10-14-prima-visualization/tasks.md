## 1. Calculation Functions

- [x] 1.1 Add `semesters(year)` helper that returns the two semester objects `{ start, end, dueDate, label }` for a given year
- [x] 1.2 Add `calcPrimaDays(emp, semester)` that computes days worked using the 30-days/month convention, prorated from `contract_start` and capped at today when semester is in progress
- [x] 1.3 Add `calcPrima(emp, semester)` that applies the formula `(monthlySalary + transportBase) × days / 360` using `getEffectiveSalary` for the salary lookup

## 2. Prima View Component

- [x] 2.1 Create `PrimaView` React component that receives the active `emp` and renders a "Prima de Servicios" card
- [x] 2.2 Render one row per semester with: semester label, period, days worked, monthly salary used, prima amount (COP), and due date
- [x] 2.3 Skip semesters where the employee has 0 days worked (hired after semester end)
- [x] 2.4 Label in-progress prima as "Acumulado al [date]" and completed as "Total"
- [x] 2.5 Highlight due date in red when ≤ 30 calendar days away and semester not yet ended

## 3. Integration

- [x] 3.1 Render `PrimaView` inside the Payroll tab, below the existing quincena calculator section
