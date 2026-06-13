## Purpose

Specifies the employee dashboard view that surfaces the next upcoming payment obligation as a hero element, provides snapshot stats (vacaciones balance and current salary), renders the full obligations agenda list with mark-as-paid actions, and retains recent absence history.

## Requirements

### Requirement: Próximo pago hero
The dashboard SHALL display, as its primary element, the next upcoming obligation: its amount (formatted COP), its label, its due date, and a countdown indicator. The countdown SHALL be visually emphasized as the due date nears.

#### Scenario: Next obligation shown
- **WHEN** the employee has at least one upcoming obligation
- **THEN** the hero shows that obligation's amount, label, due date, and a "en N días" / "mañana" / "hoy" countdown

#### Scenario: Countdown urgency
- **WHEN** the next obligation is due in 3 or fewer days
- **THEN** the countdown is shown with an urgent (red) emphasis

#### Scenario: No upcoming obligations
- **WHEN** there are no upcoming obligations
- **THEN** the hero shows a "Sin obligaciones próximas" state instead of an amount

### Requirement: Snapshot stats
The dashboard SHALL show a 2-column snapshot row with two stats: (1) vacaciones balance with availability label and next-accrual countdown as subtitle, and (2) current monthly salary with salary type (Mínimo / Personalizado) as subtitle. The UI SHALL use the Spanish term "Vacaciones", not "PTO".

#### Scenario: Stats rendered
- **WHEN** the dashboard loads
- **THEN** it shows vacaciones balance with "Disponible/Bajo/Crítico · acumula en Nd" subtitle, and monthly salary with "Mínimo YYYY / Personalizado" subtitle

#### Scenario: Vacaciones language
- **WHEN** any label referring to paid time off is displayed
- **THEN** it reads "Vacaciones" or "vacs.", never "PTO"

### Requirement: Obligations agenda list
The dashboard SHALL render the upcoming-obligations list, each row showing a type indicator, label, due date, period, countdown, and amount. Each row SHALL offer a mark-as-paid action.

#### Scenario: Agenda rendered
- **WHEN** there are upcoming obligations
- **THEN** each is shown as a row with its type color, label, date, countdown, and amount

#### Scenario: Mark as paid
- **WHEN** the user activates the mark-as-paid action on a row
- **THEN** the obligation is recorded as paid and removed from the agenda

#### Scenario: Empty agenda
- **WHEN** there are no upcoming obligations
- **THEN** the list shows an empty-state message

### Requirement: Recent absences retained
The dashboard SHALL continue to show the most recent PTO absences below the obligations agenda.

#### Scenario: Recent absences shown
- **WHEN** the employee has logged absences
- **THEN** the dashboard shows the most recent ones (up to 4)
