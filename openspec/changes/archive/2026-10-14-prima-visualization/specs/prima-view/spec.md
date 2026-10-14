## ADDED Requirements

### Requirement: Prima panel placement
The system SHALL render a prima panel in the Payroll tab, below the existing quincena calculator, for the currently active employee.

#### Scenario: Panel visible in Payroll tab
- **WHEN** the user navigates to the Payroll tab
- **THEN** a "Prima de Servicios" section is visible below the quincena calculator

### Requirement: Semester rows
The system SHALL display one row per semester of the current calendar year for which the employee has worked at least 1 day. Each row SHALL show: semester label, period dates, days worked, monthly salary used, prima amount (formatted as COP currency), and due date.

#### Scenario: Both semesters shown for established employee
- **WHEN** the active employee's contract_start is before January 1 of the current year
- **THEN** two rows are shown: one for the 1st semester and one for the 2nd semester

#### Scenario: Only future-eligible semester shown for new hire
- **WHEN** an employee was hired on August 1
- **THEN** only the 2nd semester row is shown (1st semester has 0 days worked)

### Requirement: In-progress vs. completed labeling
The system SHALL label a semester row as "Acumulado" (with today's accrued amount) when the semester is still in progress, and as "Total" when the semester has ended.

#### Scenario: Current semester shows accrued label
- **WHEN** today falls within a semester
- **THEN** the prima amount is labeled "Acumulado al [today's date]"

#### Scenario: Past semester shows total label
- **WHEN** a semester has fully ended
- **THEN** the prima amount is labeled "Total"

### Requirement: Due date urgency highlight
The system SHALL highlight the due date in red when it is ≤ 30 calendar days away and the semester has not yet ended.

#### Scenario: Due date imminent
- **WHEN** today is June 5 and the 1st semester due date is June 30
- **THEN** the due date is displayed in red

#### Scenario: Due date not imminent
- **WHEN** the due date is more than 30 days away
- **THEN** the due date is displayed in normal (non-red) style
