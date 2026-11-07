# Mi Nómina

Simple HR tracker for small teams in Colombia. Tracks PTO balances and computes semi-monthly payroll (quincena) based on 2026 minimum wage rules.

## How to use

Double-click `index.html` — opens in any browser. No installs, no server.

> **Privacy:** all data is stored exclusively in your browser's `localStorage`. Nothing is sent to any server. Use the **Export** button to back up your data as a local JSON file.

## Use on iPhone (installable web app)

The app can be installed on an iPhone from a static HTTPS host (the service worker and install prompt need HTTPS; `file://` does not support them).

1. Open the hosted URL in **Safari** (not an in-app browser).
2. Tap **Share → Agregar a inicio**. The app then launches full-screen with its own icon and works offline.
3. **Move your data over.** The installed app has its own storage, separate from Safari tabs and from your computer. On the computer tap **Exportar**, send the file to the phone (AirDrop / Files), then in the installed app tap **Importar** and choose it.

Notes:
- Import **replaces** all data on that device; it does not merge. Pick one main device, and export before switching.
- Export on iPhone opens the share sheet (Save to Files, AirDrop, …). The app reminds you to export if your last backup is over 30 days old.
- To try it locally: `python3 -m http.server`, then open `http://localhost:8000` (localhost counts as secure for the service worker).
- After every deploy, bump `CACHE_VERSION` in `sw.js`; installed apps show a "Nueva versión disponible" banner.

## Features

- **Multi-employee** — enroll employees with name, ID, phone, email, contract start date, and salary
- **PTO tracking** — accrues 1.25 days/month (15 days/year), logs absences, shows running balance
- **Payroll calculator** — pick any quincena to see gross pay, transport subsidy, social security deduction, and net pay
- **Prima de servicios** — select "Prima" in the período dropdown for June or December to see the projected semester prima per employee (prorated from contract start date)
- **Salary history** — log dated salary changes per employee; payroll auto-uses the salary in effect for each period
- **Minimum wage toggle** — set salary to "Mínimo 2026" and all figures compute automatically
- **Import / Export** — data saved in browser localStorage; Export saves a dated `mi_nomina_YYYY-MM-DD.json` (share sheet on iPhone), Import validates the file and asks before replacing your data

## Payroll rules (Colombia 2026)

| Concept | Value |
|---|---|
| Monthly minimum wage | $1,750,905 COP |
| Transport subsidy | $249,095 COP / month |
| Salary per quincena | $875,453 COP |
| Transport per quincena | $124,548 COP |
| Transport deduction per vacation day | $8,303 COP |
| Social security (employee) | 8% of salary (health 4% + pension 4%) |

Transport subsidy applies only to employees earning ≤ 2× minimum wage.

## Prima de servicios rules (Colombia 2026)

Formula: `(monthly_salary + transport_subsidy_if_eligible) × months_worked × 2.5 / 30`

| Semester | Period | Due date |
|---|---|---|
| 1st | January 1 – June 30 | June 30 |
| 2nd | July 1 – December 31 | December 20 |

Always computed to the full semester end date regardless of payment date. Prorated from `contract_start` for employees hired mid-semester.

## Updating for a new year

Two constants at the top of `index.html`:

```js
const MONTHLY_MIN_WAGE  = 1750905;  // update each January
const MONTHLY_TRANSPORT = 249095;   // update each January
```

Also add the new year to the quincena year dropdown (search for `2025,2026,2027`), and bump `CACHE_VERSION` in `sw.js` when you deploy so installed phones pick up the new constants.

## Data format

```json
{
  "active": "emp_xxx",
  "employees": [
    {
      "id": "emp_xxx",
      "name": "...",
      "last_name": "...",
      "cedula": "...",
      "phone": "...",
      "email": "...",
      "contract_start": "YYYY-MM-DD",
      "salary_history": [
        { "date": "YYYY-MM-DD", "amount": 1750905, "is_minimum": true, "note": "Ingreso" }
      ],
      "initial_balance": 9,
      "initial_date": "YYYY-MM-DD",
      "pto_events": [
        { "date": "YYYY-MM-DD", "days": 1.0, "comment": "..." }
      ],
      "paid_obligations": [
        { "type": "quincena", "key": "quincena_2026-06-15", "amount": 1124548, "paidDate": "2026-06-15" }
      ]
    }
  ]
}
```
