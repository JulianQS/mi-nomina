## Purpose

Specifies the logic for producing a unified, forward-looking list of employer payment obligations (quincenas, prima, cesantías, intereses) for a given employee, including how paid obligations are recorded, persisted, and excluded from the agenda.

## Requirements

### Requirement: Merged obligation list
The system SHALL produce a single list of upcoming employer obligations for an employee, merging quincena payments, prima de servicios, cesantías, and intereses sobre cesantías. Each item SHALL carry a `type`, a human-readable `label`, a `sub` descriptor (period), a due `date`, and an `amount`.

#### Scenario: Quincena obligations generated
- **WHEN** the agenda is generated for an active employee
- **THEN** it includes quincena pay dates (the 15th and the month-end) for upcoming months, each with its net amount from `calcPayroll`

#### Scenario: Prima obligations generated
- **WHEN** a prima due date (June 30 or December 20) is in the future and the projected prima amount is greater than 0
- **THEN** the agenda includes a prima item with that amount and due date

#### Scenario: Cesantías obligations generated
- **WHEN** the cesantías and intereses due dates for a completed year are in the future and the computed amounts are greater than 0
- **THEN** the agenda includes separate intereses (Jan 31) and cesantías (Feb 14) items

### Requirement: Forward-looking and sorted
The system SHALL exclude obligations whose due date is before today and SHALL return the remaining items sorted by due date ascending, limited to a caller-specified count.

#### Scenario: Past obligations excluded
- **WHEN** a quincena pay date has already passed
- **THEN** it does not appear in the agenda

#### Scenario: Sorted by date
- **WHEN** multiple obligations are upcoming
- **THEN** they are ordered from soonest to latest due date

#### Scenario: Obligations before contract start excluded
- **WHEN** a candidate obligation's period ends before the employee's contract_start
- **THEN** it is not included

### Requirement: Paid obligation record
The system SHALL support recording an obligation as paid. A paid record SHALL uniquely identify the obligation by `type` and a stable `period key` (e.g., type + due date), and SHALL store the `amount` and the `date paid`. Paid records SHALL persist with the employee data in `localStorage`.

#### Scenario: Recording a payment
- **WHEN** an obligation is marked as paid
- **THEN** a paid record with its type, period key, amount, and paid date is stored on the employee

#### Scenario: Backward compatibility
- **WHEN** employee data has no paid-obligations field
- **THEN** the system treats it as no obligations paid and behaves normally

### Requirement: Paid obligations excluded from agenda
The system SHALL exclude any obligation that has a matching paid record from the generated agenda, while preserving the paid record as viewable history.

#### Scenario: Paid obligation drops off
- **WHEN** an upcoming obligation has a matching paid record
- **THEN** it no longer appears in the agenda

#### Scenario: Paid history retained
- **WHEN** an obligation has been marked paid
- **THEN** its paid record remains available to display as history
