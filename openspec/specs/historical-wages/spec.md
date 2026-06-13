## Purpose

Specifies how the system maintains and resolves year-specific Colombian statutory minimum wages and transport subsidies, so that payroll and benefit calculations for historical periods use the correct legal values rather than the current-year global constants.

## Requirements

### Requirement: Wage history table
The system SHALL maintain a `WAGE_HISTORY` constant mapping each calendar year to its Colombian statutory minimum wage (`minWage`) and transport subsidy (`transport`). Years outside the table SHALL fall back to the most recent year in the table.

#### Scenario: Known year resolved
- **WHEN** wages are requested for year 2024
- **THEN** the system returns minWage = 1,300,000 and transport = 162,000

#### Scenario: Known year 2025 resolved
- **WHEN** wages are requested for year 2025
- **THEN** the system returns minWage = 1,423,500 and transport = 200,000

#### Scenario: Unknown future year falls back
- **WHEN** wages are requested for a year not yet in the table
- **THEN** the system returns the values for the most recent year in the table

### Requirement: Year-aware effective salary
`getEffectiveSalary(emp, dateStr)` SHALL resolve the minimum wage and transport subsidy from `WAGE_HISTORY` using the year of `dateStr`, not from global constants. The `is_minimum` flag SHALL use the year-appropriate minimum wage. The transport-eligibility check (`≤ 2× minimum wage`) SHALL also use the year-appropriate minimum wage. The resolved transport subsidy SHALL be returned as `transportMonthly` on the result object so callers do not need to look it up separately.

#### Scenario: Minimum-wage employee in 2024
- **WHEN** an employee has `is_minimum: true` and salary is evaluated at a 2024 date
- **THEN** `monthly` equals 1,300,000 and `transportMonthly` equals 162,000

#### Scenario: Minimum-wage employee in 2025
- **WHEN** an employee has `is_minimum: true` and salary is evaluated at a 2025 date
- **THEN** `monthly` equals 1,423,500 and `transportMonthly` equals 200,000

#### Scenario: Transport eligibility uses period minimum wage
- **WHEN** an employee's salary is evaluated for a given year
- **THEN** `has_transport` is true if and only if `monthly ≤ 2 × minWage` for that year

### Requirement: Year-aware payroll and benefit calculations
`calcPayroll`, `calcPrima`, and `calcCesantias` SHALL use `sal.transportMonthly` (from `getEffectiveSalary`) for transport amounts, never the global `MONTHLY_TRANSPORT` constant. This ensures all calculations automatically reflect the statutory transport subsidy for the period being computed.

#### Scenario: 2024 quincena transport
- **WHEN** a quincena is computed for a 2024 period for a minimum-wage employee
- **THEN** transport per quincena is based on 162,000/month, not the 2026 value

#### Scenario: Reference table matches selected year
- **WHEN** the user views the "Valores fijos" reference table in the Quincena tab for year 2025
- **THEN** the table shows minWage = 1,423,500 and transport = 200,000
