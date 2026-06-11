## Purpose

UI component that displays prima de servicios for the active employee, accessible from the Payroll tab.

## Requirements

### Requirement: Prima panel placement
The system SHALL expose prima as a "Prima" option in the Período dropdown of the Payroll tab, available only when the selected month is June or December. Selecting it replaces the quincena breakdown with the prima card for the relevant semester (June → 1st semester, December → 2nd semester). Switching away from June or December resets the selector to 1ª quincena.

#### Scenario: Prima option appears for June
- **WHEN** the user selects June in the Payroll tab
- **THEN** the Período dropdown includes a "Prima" option alongside 1ª and 2ª

#### Scenario: Prima option absent for other months
- **WHEN** the user selects any month other than June or December
- **THEN** the Período dropdown shows only 1ª and 2ª options

#### Scenario: Selecting Prima shows prima card
- **WHEN** the user selects "Prima" in the Período dropdown
- **THEN** the quincena breakdown is replaced by the Prima de Servicios card

### Requirement: Semester rows
The system SHALL display one row per semester for which the employee has worked at least 1 day. Each row SHALL show: semester label, period dates, months worked (with "2.5 días/mes" rate), monthly salary used, prima amount (formatted as COP currency), and due date.

#### Scenario: Both semesters shown for established employee
- **WHEN** the active employee's contract_start is before January 1 of the current year
- **THEN** two rows are shown: one for the 1st semester and one for the 2nd semester

#### Scenario: Only eligible semester shown for new hire
- **WHEN** an employee was hired on August 1
- **THEN** only the 2nd semester row is shown (1st semester has 0 days worked)

### Requirement: In-progress vs. completed labeling
The system SHALL label a semester row as "Proyectado al cierre del semestre" when the semester is still in progress, and as "Total" when the semester has ended.

#### Scenario: Current semester shows projected label
- **WHEN** today falls within a semester
- **THEN** the prima amount is labeled "Proyectado al cierre del semestre"

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
