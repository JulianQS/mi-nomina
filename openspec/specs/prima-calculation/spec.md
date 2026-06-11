## Purpose

Compute prima de servicios per employee per semester according to Colombian labor law (CST Art. 306). Prima is always projected to the full semester end date regardless of today's date.

## Requirements

### Requirement: Semester bounds
The system SHALL define two fixed semesters per calendar year:
- 1st semester: January 1 – June 30, due June 30.
- 2nd semester: July 1 – December 31, due December 20.

#### Scenario: First semester bounds
- **WHEN** year is 2026
- **THEN** 1st semester start is 2026-01-01, end is 2026-06-30, due date is 2026-06-30

#### Scenario: Second semester bounds
- **WHEN** year is 2026
- **THEN** 2nd semester start is 2026-07-01, end is 2026-12-31, due date is 2026-12-20

### Requirement: Days worked calculation
The system SHALL compute days worked using the 30-days/month convention, capped at 180 per semester. The start of the accrual period is `max(semesterStart, contractStart)`. The end is always `semesterEnd`, regardless of today's date — the prima is always projected to the full semester close, since payment typically happens before the deadline.

#### Scenario: Full semester
- **WHEN** an employee's contract_start is before the semester start
- **THEN** days worked equals 180

#### Scenario: Employee hired mid-semester
- **WHEN** an employee's contract_start falls within the semester (e.g., February 15 for the 1st semester)
- **THEN** days worked is prorated from contract_start to semester end using 30-day months

#### Scenario: Semester in progress
- **WHEN** today falls within the semester
- **THEN** days worked is still calculated to semesterEnd (projected), labeled "Proyectado al cierre del semestre"

### Requirement: Prima amount formula
The system SHALL compute prima as: `Math.round((monthlySalary + transportBase) × daysWorked / 360)`, where `transportBase` is `MONTHLY_TRANSPORT` if the employee earns ≤ 2× `MONTHLY_MIN_WAGE`, otherwise 0. `monthlySalary` is derived from `getEffectiveSalary(emp, semesterEndDate).monthly`.

#### Scenario: Full semester at minimum wage
- **WHEN** an employee earned minimum wage (1,750,905 COP) for the full 1st semester and is eligible for transport (249,095 COP)
- **THEN** prima = round((1,750,905 + 249,095) × 180 / 360) = 1,000,000 COP

#### Scenario: Employee earning above 2× minimum wage
- **WHEN** an employee earns 4,000,000 COP per month (above 2× minimum wage)
- **THEN** transport subsidy is NOT included in prima base

#### Scenario: Zero days worked
- **WHEN** an employee's contract_start is after the semester end
- **THEN** prima is 0 and the semester is not shown
