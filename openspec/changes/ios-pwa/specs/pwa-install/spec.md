## ADDED Requirements

### Requirement: Web app manifest
The app SHALL ship a web app manifest at `manifest.webmanifest`, linked from `index.html`, declaring name "Mi Nómina", short name "Nómina", `lang` "es-CO", `start_url` and `scope` relative to the app directory, `display` "standalone", a theme color matching the header (indigo-700, `#4338ca`), a background color matching the page (`#f1f5f9`), and PNG icons at 192×192 and 512×512 (including a maskable variant).

#### Scenario: Manifest is discoverable
- **WHEN** `index.html` is loaded over HTTPS
- **THEN** the document head contains `<link rel="manifest" href="manifest.webmanifest">` and the file parses as a valid manifest with the fields above

#### Scenario: Manifest not ignored by git
- **WHEN** the manifest is added to the repository
- **THEN** it is tracked by git (its extension is not matched by the `*.json` ignore rule)

### Requirement: iOS home-screen metadata
`index.html` SHALL include an `apple-touch-icon` (180×180 PNG), `apple-mobile-web-app-title` set to "Mi Nómina", `apple-mobile-web-app-capable` and `mobile-web-app-capable` set to "yes", an `apple-mobile-web-app-status-bar-style` of "black-translucent", and a `theme-color` meta tag.

#### Scenario: Add to Home Screen on iPhone
- **WHEN** a user opens the hosted app in Safari on iOS and taps Share → Add to Home Screen
- **THEN** the suggested name is "Mi Nómina" and the home-screen icon is the app icon, not a page screenshot

#### Scenario: Launch from home screen
- **WHEN** the user launches the app from its home-screen icon
- **THEN** it opens full-screen without Safari's address bar or toolbar, showing the app's own header and bottom tab bar

### Requirement: Install guidance
The app SHALL show a dismissible hint explaining how to install it ("Compartir → Agregar a inicio") when it is running in Safari on iOS and not in standalone mode. Once dismissed, the hint SHALL NOT reappear on that device.

#### Scenario: Hint in Safari tab
- **WHEN** the app is opened in a regular Safari tab on an iPhone and the hint has not been dismissed
- **THEN** a short install hint is displayed

#### Scenario: No hint when installed
- **WHEN** the app is running in standalone mode (`navigator.standalone` or `display-mode: standalone`)
- **THEN** no install hint is shown

#### Scenario: No hint on desktop
- **WHEN** the app is opened in a desktop browser
- **THEN** no install hint is shown
