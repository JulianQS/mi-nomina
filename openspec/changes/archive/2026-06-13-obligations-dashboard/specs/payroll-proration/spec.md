## ADDED Requirements

### Requirement: Prorate quincena for contract start date
`calcPayroll` SHALL detect when an employee's `contract_start` falls within the quincena period and prorate both salary and transport subsidy proportionally. The proration factor is `daysWorked / 30`, where `daysWorked` is the number of calendar days from `contract_start` (or the period start, whichever is later) to the period end inclusive. Normal quincenas (contract start before the period) use the standard `monthly / 2` calculation unchanged.

#### Scenario: Contract start mid first quincena
- **WHEN** `contract_start` is September 9 and the period is September 1–15
- **THEN** `daysWorked` is 7, salary = `monthly × 7 / 30`, transport = `transportMonthly × 7 / 30`

#### Scenario: Contract start before period — no proration
- **WHEN** `contract_start` is September 9 and the period is September 16–30
- **THEN** no proration applies; salary = `monthly / 2`, transport = `transportMonthly / 2`

#### Scenario: Contract start after period end
- **WHEN** `contract_start` is after the quincena end date
- **THEN** `daysWorked` is 0 and all amounts are 0

### Requirement: Proration indicator in UI
When a quincena is prorated, the payroll breakdown SHALL display a badge indicating the proration ratio (days worked out of total days in the period), so the employer can verify the calculation.

#### Scenario: Prorated badge shown
- **WHEN** a quincena result has `prorated: true`
- **THEN** a badge reading "Prorrateado · N/M días" appears in the quincena header

#### Scenario: No badge for full quincena
- **WHEN** a quincena result has `prorated: false`
- **THEN** no proration badge is shown

### Requirement: Period dropdown reset on month change
When the selected period is a special view ("Prima" or "Cesantías") and the user switches to a month where that view is unavailable, the period SHALL automatically reset to "1ª" to prevent an invalid selection state.

#### Scenario: Cesantías resets when leaving January / February
- **WHEN** "Cesantías" is selected and the user picks any month other than 1 or 2
- **THEN** the period resets to "1ª"

#### Scenario: Prima resets when leaving June / December
- **WHEN** "Prima" is selected and the user picks any month other than 6 or 12
- **THEN** the period resets to "1ª"
