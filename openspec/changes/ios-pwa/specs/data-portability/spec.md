## ADDED Requirements

### Requirement: Export via share sheet
"Exportar" SHALL produce a JSON file named `mi_nomina_YYYY-MM-DD.json` containing the full database. When the Web Share API supports sharing files (`navigator.canShare({ files })`), the app SHALL open the system share sheet with that file; otherwise it SHALL fall back to a browser download. Cancelling the share sheet SHALL NOT show an error.

#### Scenario: Export on iPhone
- **WHEN** the user taps "Exportar" in the installed app on iOS
- **THEN** the iOS share sheet opens with the JSON file, offering Save to Files, AirDrop and Messages

#### Scenario: Export on desktop
- **WHEN** the user clicks "Exportar" in a browser without file sharing support
- **THEN** the file downloads as before, with the dated filename

#### Scenario: Share cancelled
- **WHEN** the user dismisses the share sheet without choosing a target
- **THEN** nothing else happens and no error is shown

### Requirement: Validated import with confirmation
"Importar" SHALL accept JSON files picked from the iOS Files app. Before replacing data, the app SHALL validate that the file parses and has an `employees` array of objects each with `id`, `name` and `salary_history`, and SHALL ask the user to confirm, showing the number of employees in the file and that current data will be replaced. Invalid files SHALL be rejected with a message and leave current data untouched. The legacy `pto_db` shape SHALL be accepted and migrated the same way `loadDB` does.

#### Scenario: Valid file confirmed
- **WHEN** the user picks a valid export and confirms
- **THEN** the app's data is replaced with the file contents and the active employee is shown

#### Scenario: Valid file cancelled
- **WHEN** the user picks a valid export and cancels the confirmation
- **THEN** current data is unchanged

#### Scenario: Invalid file
- **WHEN** the user picks a file that is not JSON or lacks the `employees` array
- **THEN** the app shows "Archivo inválido" with a short reason and current data is unchanged

#### Scenario: Files app selectable
- **WHEN** the user taps "Importar" on iOS
- **THEN** `.json` files in the Files app are selectable (not greyed out)

### Requirement: Persistent storage request
On startup, when `navigator.storage.persist` is available, the app SHALL request persistent storage once. Failure or denial SHALL NOT affect app behavior.

#### Scenario: Persist supported
- **WHEN** the app starts in a browser that supports `navigator.storage.persist`
- **THEN** it requests persistence without user-visible errors

### Requirement: Backup reminder
The app SHALL record the date of the last successful export (stored outside `mi_nomina_db`, so it is not included in exports). When data exists and the last export is missing or older than 30 days, the app SHALL show a dismissible reminder suggesting an export. Dismissing hides it for 7 days.

#### Scenario: Never exported
- **WHEN** the user has employee data and has never exported on this device
- **THEN** a backup reminder is shown with an "Exportar ahora" action

#### Scenario: Recent export
- **WHEN** the last export was 10 days ago
- **THEN** no reminder is shown

#### Scenario: Reminder dismissed
- **WHEN** the user dismisses the reminder
- **THEN** it does not reappear for 7 days

### Requirement: First-run import prompt
When the app runs in standalone mode and the database is still the untouched demo data (`DEFAULT_DB`), the dashboard SHALL show a prompt explaining that the installed app has its own storage and offering an "Importar datos" action that opens the import file picker.

#### Scenario: Fresh install
- **WHEN** the user launches the newly installed app for the first time
- **THEN** an "Importar datos" prompt is shown above the dashboard

#### Scenario: Real data present
- **WHEN** the database contains data other than the demo employee
- **THEN** no first-run import prompt is shown
