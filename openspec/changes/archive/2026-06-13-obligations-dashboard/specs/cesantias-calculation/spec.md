## ADDED Requirements

### Requirement: Cesantías period and due dates
The system SHALL treat cesantías as an annual obligation for calendar year `Y` (January 1 – December 31), with statutory due dates: intereses sobre cesantías paid to the employee by January 31 of year `Y+1`, and cesantías deposited to a fund by February 14 of year `Y+1`.

#### Scenario: Due dates for a calendar year
- **WHEN** the year is 2025
- **THEN** the intereses due date is 2026-01-31 and the cesantías due date is 2026-02-14

### Requirement: Days worked calculation
The system SHALL compute days worked using the 30-days/month convention, capped at 360 per year. The accrual start is `max(yearStart, contractStart)` and the end is always `yearEnd` (December 31).

#### Scenario: Full year worked
- **WHEN** an employee's contract_start is on or before January 1 of the year
- **THEN** days worked equals 360

#### Scenario: Employee hired mid-year
- **WHEN** an employee's contract_start falls within the year (e.g., September 9)
- **THEN** days worked is prorated from contract_start to December 31 using 30-day months

#### Scenario: Contract starts after the year ends
- **WHEN** an employee's contract_start is after December 31 of the year
- **THEN** cesantías and intereses are both 0

### Requirement: Cesantías amount formula
The system SHALL compute cesantías as `Math.round((monthlySalary + transportBase) × daysWorked / 360)`, where `transportBase` is `MONTHLY_TRANSPORT` if the employee earns ≤ 2× `MONTHLY_MIN_WAGE` (otherwise 0), and `monthlySalary` is `getEffectiveSalary(emp, yearEndDate).monthly`.

#### Scenario: Full year at minimum wage
- **WHEN** an employee earned minimum wage (1,750,905 COP) the full year and is eligible for transport (249,095 COP)
- **THEN** cesantías = round((1,750,905 + 249,095) × 360 / 360) = 2,000,000 COP

#### Scenario: Employee earning above 2× minimum wage
- **WHEN** an employee earns above 2× minimum wage
- **THEN** transport subsidy is NOT included in the cesantías base

### Requirement: Intereses sobre cesantías formula
The system SHALL compute intereses as `Math.round(cesantias × daysWorked × 0.12 / 360)`.

#### Scenario: Full year intereses
- **WHEN** cesantías equals 2,000,000 COP and days worked equals 360
- **THEN** intereses = round(2,000,000 × 360 × 0.12 / 360) = 240,000 COP

#### Scenario: Partial year intereses
- **WHEN** days worked is less than 360
- **THEN** intereses scales proportionally with days worked

### Requirement: Cesantías period view in Quincena tab
When the selected month is January or February, the period dropdown in the Quincena tab SHALL expose a "Cesantías" option. Selecting it replaces the quincena breakdown with `CesantiasAnualView` — a full calculation card for the prior calendar year (`year - 1`), mirroring the depth of the Prima view used for June and December. Switching to a different month while "Cesantías" is selected SHALL automatically reset the period to "1ª".

#### Scenario: Period option available in January / February
- **WHEN** the user selects January or February in the Quincena tab
- **THEN** the period dropdown includes a "Cesantías" option in addition to "1ª" and "2ª"

#### Scenario: Period option absent in other months
- **WHEN** the user selects any month other than January or February
- **THEN** the period dropdown does NOT include "Cesantías"; if it was previously selected it resets to "1ª"

#### Scenario: CesantiasAnualView shown
- **WHEN** the user selects "Cesantías" for January of year Y
- **THEN** `CesantiasAnualView` renders for year Y-1, showing intereses (due Jan 31, Y), cesantías deposit (due Feb 14, Y), formula strings, and a "Base del cálculo" detail table

#### Scenario: Urgency highlighting
- **WHEN** a due date is 30 or fewer days away
- **THEN** its due-date label is shown in red

#### Scenario: Employee not yet started in prior year
- **WHEN** the computed cesantías for year Y-1 is 0 (employee had not started)
- **THEN** `CesantiasAnualView` shows a neutral message ("Sin cesantías para Y-1") rather than an empty calculation card
